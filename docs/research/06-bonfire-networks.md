# Deep Dive Report 06: Bonfire Networks — Community-First Federated Social Infrastructure

> **Series:** Mycelium Research Reports — Federated Agent Orchestration on Decentralized Social Protocols
> **Report:** 06 — Bonfire Networks
> **Status:** Research Complete
> **Sources:** See [Sources & References](#sources--references) section below

---

## Executive Summary

Bonfire is both a federated social network and a modular framework for building custom applications on the open social web. Launched as Bonfire Social 1.0 in November 2025 alongside a crowdfunding campaign, it represents a distinct philosophy in the fediverse landscape: rather than building another Mastodon clone, Bonfire provides composable building blocks that communities can assemble into purpose-specific federated applications — from microblogging to project coordination to open science collaboration.

Where Mastodon asks "how do we replicate Twitter but decentralized?", Bonfire asks a deeper question: **how do we build digital infrastructure where communities govern themselves, privacy is granular and consent-based, and the platform adapts to the community rather than the other way around?**

Funded by NLNet and developed as free software, Bonfire's vision is that "the social web should be open and portable — profiles, relationships, and data move with you, governance lives with communities." For the Mycelium project, Bonfire's architecture offers one of the most complete reference implementations of community-governed federated infrastructure in the ecosystem.

---

## Overview: What Is Bonfire?

Bonfire resists a single definition, and its creators consider this a feature. Different communities describe it differently:

- **A Mastodon alternative** — a federated social network with richer governance and privacy controls
- **A framework** — composable building blocks ("fediverse legos") for constructing custom federated apps
- **A social-networking-focused Nextcloud** — modular, self-hosted, extensible
- **Community infrastructure** — tools for collective organizing, mutual aid, and participatory governance

At its core, Bonfire is two things simultaneously:

1. **Bonfire Social** — a ready-to-deploy federated social application with microblogging, feeds, groups, messaging, and sophisticated privacy/moderation controls
2. **The Bonfire Framework** — an Elixir-based toolkit for building any kind of federated social application, with ActivityPub baked in from the foundation

The 1.0 launch in November 2025 included comprehensive documentation, developer resources, and a crowdfunding campaign designed to give communities a direct voice in the platform's development roadmap. This approach reflects the operational philosophy: rather than a centralized company making unilateral product decisions, Bonfire involves communities in funding and feature prioritization.

---

## Technical Architecture

### Stack: Elixir, Phoenix, and PostgreSQL

Bonfire is built entirely in **Elixir** on the **Phoenix** web framework, with **PostgreSQL** as the relational database layer. Each of these choices carries architectural significance.

**Why Elixir?** Elixir runs on the Erlang virtual machine (BEAM), a runtime originally designed for telecom systems requiring near-zero downtime. This provides:

- **Lightweight processes**: The BEAM VM manages millions of concurrent lightweight processes. Each connected user, each federation delivery, each webhook — all run as isolated processes that cannot crash each other.
- **Fault tolerance**: OTP (Open Telecom Platform) supervisory trees automatically restart failed processes. If an ActivityPub delivery handler crashes, the supervisor restarts it without affecting other operations.
- **Hot code reloading**: Updates can be applied without stopping the system — critical for infrastructure that communities depend on for coordination.
- **Functional programming**: Elixir's immutability and pattern matching reduce entire classes of bugs common in mutable-state systems. Data flows through transformation pipelines rather than being mutated in place.

For federated social infrastructure that must handle concurrent connections, process incoming activities from dozens of remote servers simultaneously, and remain available for community coordination, Elixir/BEAM is an exceptionally well-matched choice.

**Phoenix** provides the HTTP layer, routing, and controller logic. Bonfire extends Phoenix with **Surface**, which provides an alternative syntax for LiveView components designed to be more accessible to frontend developers while adding compile-time checks. Surface compiles to LiveView code at build time — it is a developer-experience abstraction with zero runtime cost.

**PostgreSQL** handles persistent storage. The Ecto library provides the abstraction between Elixir code and the database, with schemas defined in code while leveraging PostgreSQL's relational power — transactions, foreign keys, and advanced query capabilities well-suited to storing complex social graph data: users, posts, activities, relationships, and permissions.

### Modular Design: Extensions, Flavours, and Deployment Profiles

The most distinctive architectural element of Bonfire is its extension system. The "core" Bonfire is remarkably small — just the basic application logic and Phoenix configuration. **Nearly everything else is an extension.**

This is not a superficial plugin system. Extensions are first-class citizens that follow the same patterns as core code:

- **Extensions implement their own database schemas**, migrations, context modules, UI components, and routes
- **Extensions exist in a dependency tree** — they can depend on other extensions
- **Extensions are independently toggleable** — disabled extensions contribute no routes, no UI elements, no database queries
- **Extensions are versioned as Git repositories** — communities can fork, modify, and contribute back

Foundational extensions that many others depend on include:

| Extension | Responsibility |
|-----------|---------------|
| `bonfire_common` | Suite of shared helpers used across the ecosystem |
| `bonfire_boundaries` | ACLs, circles, granular permission checking |
| `bonfire_social` | Feed activities, activity streams, threads |
| `bonfire_posts` | Content creation and storage |
| `bonfire_tag` | Tagging, mentions, hashtags |
| `bonfire_federation` | ActivityPub protocol implementation |

Beyond these, the extension ecosystem includes: `bonfire_valueflows` (economic activities using the ValueFlows vocabulary), `bonfire_gatherings` (events), `bonfire_messages` (direct messaging), `bonfire_pages` (static content), `bonfire_notify` (push notifications), and many more.

**Flavours** are curated bundles of extensions with appropriate defaults, forming deployment profiles:

- **Social** — microblogging, feeds, groups (the "Mastodon alternative" profile)
- **Coordination** — Kanban boards, task management, ValueFlows integration for tracking work and resources
- **Open Science** — publication sharing, peer review workflows, ORCID/CrossRef integration, DOI generation
- **Community** — group spaces, collective moderation, decision-making tools for neighborhood groups and organizations

A community deploys the flavour matching their needs, then enables or disables individual extensions to fine-tune the experience. This is fundamentally different from forking: communities stay on the upgrade path while customizing their deployment.

---

## ActivityPub Integration: Federation as Foundation

Bonfire's approach to ActivityPub is architecturally distinctive. **ActivityPub is not bolted onto Bonfire as an afterthought — it is foundational to how data is structured and operations are designed.**

This is not a minor distinction. Many federated platforms begin with a monolithic application and add federation support later, resulting in impedance mismatches where the federation layer must translate between internal data models and ActivityPub's object model. Bonfire begins with ActivityPub and ActivityStreams as the schema language. Internal objects are designed to be serializable to ActivityPub formats with minimal translation.

### Protocol Implementation

Bonfire implements the full ActivityPub server-to-server (S2S) specification:

- **Inbox/Outbox pattern**: Each actor exposes standard endpoints (`/pub/actors/{username}/inbox`, `/pub/actors/{username}/outbox`) for receiving and publishing activities
- **HTTP Signature verification**: All server-to-server requests are cryptographically signed and verified
- **WebFinger discovery**: Translates human-readable identifiers (`@alice@example.com`) into canonical ActivityPub actor URIs
- **Multiple actor types**: Users, organizations, groups, collections — not limited to individual user accounts
- **Full activity type support**: Create, Update, Delete, Follow, Accept, Reject, Undo, Like, Announce, Flag
- **Broad object type support**: Note, Article, Image, Video, Audio, Event — with graceful fallback display for unrecognized types
- **ULIDs for object IDs**: Universally Unique Lexicographically Sortable Identifiers ensure uniqueness and consistent ordering across the distributed network

### Federation Flow

When a user creates a post:

1. The **boundaries system** determines whether the content should be federated at all (users can opt out of federation entirely)
2. The post is serialized into ActivityStreams JSON with proper addressing fields (`to`, `cc`, `bcc`)
3. The **boundary system determines the exact audience** — not just "all followers" but the specific circles that should receive this content
4. Activities are delivered only to appropriate inboxes on remote servers

On the receiving end:

1. The activity arrives at the inbox endpoint
2. **HTTP signature is validated** (cryptographic verification of the sending server)
3. The activity enters the **ingestion pipeline** — validation, authorization checking
4. **Local extensions process the activity** — creating records, adding to feeds, triggering notifications, applying moderation policies, enforcing boundaries

### Beyond Status Updates

Crucially, Bonfire's federation is **not limited to status updates**. The extension architecture means that any new ActivityPub activity type can be federated. Economic activities via ValueFlows, event coordination via gatherings, collaborative editing — all can flow through the same federation layer. The system is designed to gracefully handle novel object types from other fediverse platforms, storing and previewing them even when not explicitly supported.

Bonfire also implements experimental Federation Enhancement Proposals (FEPs), most notably **FEP-044f for interaction policies** — allowing post authors to specify what interactions they permit, even when content is federated to remote servers.

---

## The Boundaries and Consent System

Bonfire's boundaries system is its most sophisticated and distinctive feature — a **fundamental rethinking of how privacy should work in social networks** that goes far beyond the familiar public/followers-only/private trichotomy.

### Beyond Binary Privacy Models

Mastodon offers four visibility levels: public, unlisted, followers-only, and direct. This maps poorly to actual human social dynamics. You might want to share something with close friends but not family. You might want a post visible to everyone but only allow replies from people you trust. You might want to share within a specific project team across multiple instances.

Bonfire's boundaries framework provides granular control through four interlocking concepts:

| Concept | Description |
|---------|-------------|
| **Circles** | User-defined groups of people (friends, family, colleagues, project teams). Per-user — your groupings are private, others are not notified of placement. |
| **ACLs** | Access Control Lists — collections of grants that define permissions for specific objects |
| **Grants** | Associations between a circle (or individual user), a set of permissions, and a specific ACL |
| **Verbs** | Specific permissions that can be granted or denied: read, interact, participate, contribute, moderate, caretake |

### Role-Based Permissions

Permissions are organized into roles that group related verbs:

- **Read** — see the object in feeds and searches
- **Interact** — like, follow, bookmark
- **Participate** — reply, mention, engage in discussion
- **Contribute** — create new related objects
- **Moderate** — review and act on flags/reports
- **Caretake** — deletion and major modifications

Critically, the system also includes **negative roles**:

- **Cannot Read** — object does not appear in feeds/searches; all interactions blocked
- **Cannot Interact** — object visible but no interaction permitted

### Conflict Resolution

When a user belongs to multiple circles with conflicting permissions, the system resolves through a conservative hierarchy: **No > Yes > Neutral**. Explicit denial always overrides permission. If a user is in both a "trusted friends" circle (full interaction) and a "restricted" circle (no interaction), their actual permission defaults to restriction. This ensures permissions are never accidentally granted through overlapping grants.

### Consent-Based Interactions

The boundaries system enables **consent-based interaction patterns**:

- A post can be visible to everyone but only allow replies from specific circles
- Quote-posting can require **approval from the original author** before the quote is created or federated
- Federation itself can be conditional — boundaries determine whether content leaves the local instance at all

Through FEP-044f integration, these policies travel with federated content. A remote platform supporting interaction policies will enforce them; if it does not, Bonfire still blocks unauthorized interactions from reaching the local instance.

### Presets and Composability

Users can create **reusable boundary presets** ("close collaborators", "public with discussion", "project team only") and apply or customize them per-post. This makes granular privacy practical rather than tedious — most posts use a preset, with per-post customization available when needed.

---

## Community Governance and Moderation

Bonfire's moderation architecture reflects the principle that **communities should be empowered to maintain safety according to their own values**, not dependent on platform-wide algorithmic enforcement.

### Multi-Layer Moderation

Moderation operates at three distinct layers:

1. **Instance-level**: Administrators set instance-wide policies, rules, and defaults. Transparent moderation logs (with configurable visibility) ensure accountability.
2. **Community-level**: Groups and communities within an instance can set their own guidelines, appoint moderators, and enforce norms specific to their context. A poetry workshop and a tech support forum within the same instance can have very different moderation standards.
3. **User-level**: Individual controls — blocks, mutes, content filtering, feed customization — give users agency independent of instance or community moderation.

### Federated Moderation Toolkit

Bonfire is developing the **Federated Moderation Toolkit**, an ambitious research-driven initiative funded by NLNet and designed in collaboration with 200+ experienced moderators from the IFTAS (Independent Federated Trust and Safety) community. The toolkit has three layers:

**Technical Infrastructure**: Community-informed moderation features developed through co-design with practicing moderators. Built using Bonfire's ActivityPub framework as reference implementations adoptable by any ActivityPub platform.

**Operational Framework**: Evidence-based workflows, response templates, and shared vocabulary informed by standards from the Trust and Safety Professionals Association and ISO digital safety specifications.

**Knowledge Ecosystem**: Documentation, implementation guides, and educational resources making sophisticated moderation accessible regardless of technical resources.

A key innovation is the **unified federated moderation dashboard** — allowing moderators to manage safety across multiple fediverse platforms (Mastodon, Pixelfed, Bonfire) from a single interface, coordinating responses without centralizing control.

### Decentralized Moderation Philosophy

Different spaces can have different rules. This is not a bug — it reflects the reality that a neighborhood mutual aid group, an academic research network, and a creative community have genuinely different safety needs. Bonfire provides the tooling for each to govern itself while maintaining the ability to coordinate on cross-community threats.

---

## Comparison with Mastodon

| Aspect | Bonfire | Mastodon |
|--------|---------|----------|
| **Architecture** | Modular, extension-based | Monolithic Ruby on Rails |
| **Customization** | Enable/disable extensions per instance | Fork required for deep changes |
| **Deployment** | Flavours — range from minimal to feature-rich | All-or-nothing deployment |
| **Federation** | Foundational — designed from the ground up | Added to existing application |
| **Privacy model** | Circles, ACLs, granular verbs, consent-based | 4 fixed visibility levels |
| **Governance** | Built-in participatory tools, moderation toolkit | Left to individual instances |
| **Actor types** | Users, organizations, groups, collections | Primarily individual users |
| **Content types** | Extensible via extensions and ActivityPub vocabulary | Primarily microblog posts |
| **Use cases** | Specializable (social, coordination, science) | General microblogging |
| **Language/Runtime** | Elixir/BEAM (concurrent, fault-tolerant) | Ruby/Rails (mature, widely known) |

Mastodon's strength is adoption — it has the largest fediverse user base and the most mature ecosystem. Bonfire's strength is architectural flexibility — it can become things Mastodon cannot without forking. They are complementary rather than competitive: Bonfire instances federate seamlessly with Mastodon instances.

---

## The Bonfire Framework: Building Custom Federated Applications

Beyond Bonfire Social, the framework represents a paradigm shift in social software development. Rather than isolated projects building applications that cannot interoperate, developers build on shared infrastructure — ActivityPub federation, the extension ecosystem, the boundaries system — while customizing for their specific domain.

### Development Path

1. **Generate extension scaffolding**: A mix task creates the directory structure — schemas, migrations, context modules, UI components, routes, tests
2. **Implement domain logic**: Extensions can define new database schemas, business logic, API endpoints, and UI components — or reuse existing components from foundational extensions
3. **Register routes conditionally**: Extensions link routes that only activate when the extension is enabled — clean separation, no conflicts
4. **Version as Git repository**: Extensions are versioned and distributed as Git repos — forkable, contributable, independently deployable
5. **Deploy as federated instance**: The resulting application automatically interoperates with the fediverse through the shared ActivityPub layer

### Extension Capabilities

Extensions can implement:

- **Custom database schemas and migrations** — new data models specific to the domain
- **API extensions** — GraphQL schema fields via Absinthe, custom REST endpoints
- **New ActivityPub activity types** — federate domain-specific operations
- **UI themes and custom frontends** — Surface/LiveView components, full visual customization
- **Storage backend swapping** — alternative file storage, caching layers
- **Custom authentication methods** — SSO, OAuth providers, institutional login
- **Domain-specific workflows** — collaborative editing, event organizing, reputation systems, economic coordination

### Real-World Extension Examples

- **`bonfire_valueflows`**: Implements the ValueFlows open economic vocabulary — cooperative communities can coordinate production and resource exchange with federated visibility
- **`bonfire_gatherings`**: Event coordination and scheduling
- **`bonfire_editor_milkdown`**: Rich markdown editing — available to communities that want it, absent from those that don't
- **`bonfire_invite_links`**: Configurable invitation system with usage limits and expiration
- **Open Science extensions**: Publication sharing, peer review, ORCID integration, DOI generation

The framework's power is that all of these automatically inherit federation, boundaries, governance, and moderation capabilities from the shared infrastructure.

---

## Data Portability and Migration

Bonfire implements account migration following ActivityPub standards, addressing a fundamental problem in federated networks: what happens when a user outgrows their instance?

Users can export all their data — follows, blocks, mutes, bookmarks, circles, complete post history — in standard formats (CSV, JSON). When moving to a new Bonfire instance, they import these exports, and the system re-follows accounts, re-applies blocks and mutes, recreates circles, and restores post history with preserved dates and original URLs. The import system is idempotent — it skips duplicates and handles malformed data gracefully.

An account alias mechanism prevents impersonation during migration: both old and new accounts verify the relationship cryptographically.

---

## Vision: Social Web vs. Social Media

Bonfire articulates a clear distinction between these concepts that goes beyond terminology to reflect different architectural choices:

**Social media** (the status quo): Closed systems optimizing for engagement and advertising revenue. Algorithmic feeds maximize time-on-platform. Users cannot port their content or followers. The platform owns all data. Moderation is opaque, corporate-controlled, and optimized for advertiser comfort rather than community safety.

**Social web** (Bonfire's vision): Interconnected servers exchanging information through open protocols. Users maintain control over their data. Communities implement transparent moderation with clear rules and appeals processes. Algorithms are user-controlled or community-governed. No single entity captures all value.

Bonfire positions itself explicitly as **social web infrastructure**. The project envisions a tapestry of different social spaces built with shared building blocks but diverging in purpose — one community runs a social flavour resembling Mastodon, another runs a coordination flavour resembling a project management tool, a third runs custom extensions for their specific domain. All federate with each other and with the broader fediverse.

The Cooperative Hosting Network extends this vision to infrastructure itself — drawing parallels with Community Supported Agriculture, where multiple communities collectively fund and sustain the infrastructure they depend on, with transparent value distribution among contributors.

---

## Key Takeaways for Mycelium

Bonfire's architecture offers direct insights for designing community-governed agent ecosystems. Where Mycelium asks "how do communities govern agents operating in their spaces?", Bonfire has already built many of the primitives such a system would require.

### 1. The Boundaries System as Agent Policy Infrastructure

Bonfire's Circles + ACLs + Verbs model maps almost directly to agent permission systems:

- **Circles become agent trust tiers**: A community could define circles like "trusted agents", "experimental agents", "quarantined agents" — each with different permission grants
- **Verbs become agent capabilities**: Read (observe feeds), Interact (react to content), Participate (post replies), Contribute (create new content), Moderate (flag or filter content), Caretake (modify community resources)
- **Consent-based interaction transfers to agent consent**: Just as a user can require approval before someone quotes their post, a community could require approval before an agent operates in their space
- **The No > Yes > Neutral hierarchy** provides a safe default for agent permissions — explicit denial always wins, preventing accidental capability escalation

This is precisely the kind of granular, consent-based permission model that agent ecosystems need. A community might allow a summarization agent to read all public posts but not interact; allow a moderation-assistance agent to flag content but not delete it; and block all agents from a specific developer entirely — all expressible in Bonfire's existing boundary vocabulary.

### 2. Extensions as Agent Integration Points

Bonfire's extension architecture provides a template for agent integration:

- **Agents as extensions**: An agent could be packaged as a Bonfire extension — with its own schemas (for tracking what it has processed), context modules (for its decision logic), and routes (for configuration UI)
- **Independently toggleable**: Just as communities enable/disable extensions, they could enable/disable specific agents — no instance-wide decisions required
- **Flavour-based agent bundles**: A "moderation-assisted" flavour might include flagging agents; a "research" flavour might include summarization and citation agents
- **Extension dependency trees**: Agents that depend on other agents' outputs can declare those dependencies explicitly

### 3. Federated Moderation as Agent Governance Model

The Federated Moderation Toolkit's three-layer design (technical infrastructure, operational framework, knowledge ecosystem) maps to agent governance:

- **Technical**: APIs and protocols for controlling agent behavior, reporting agent misbehavior, and coordinating across instances
- **Operational**: Standard workflows for onboarding new agents, reviewing agent outputs, escalating concerns, and revoking agent access
- **Knowledge**: Documentation and best practices for communities choosing which agents to trust

The multi-layer moderation model (instance → community → user) provides the right granularity: an instance admin might ban a known-malicious agent entirely, a community moderator might restrict an agent's capabilities within their group, and an individual user might mute all agent-generated content from their personal feed.

### 4. ActivityPub as Agent Communication Protocol

Bonfire demonstrates that ActivityPub can carry far more than microblog posts. The same infrastructure that federates economic activities (ValueFlows) and events (gatherings) could federate agent operations:

- **Agents as ActivityPub actors**: With inboxes, outboxes, and proper addressing
- **Agent actions as Activities**: Create (generate content), Flag (report issues), Announce (amplify), custom activity types for agent-specific operations
- **Federation-aware boundaries**: An agent's outputs can be scoped using the same boundary system — visible only to specific circles, requiring approval before crossing instance boundaries

### 5. The "Flavours" Model for Agent Ecosystem Diversity

Different communities have different relationships with automation. Bonfire's flavours model suggests an approach:

- **Minimal flavour**: No agents, fully human-driven — for communities that value unmediated interaction
- **Assisted flavour**: Opt-in agents for moderation support, content summarization, accessibility (alt-text generation)
- **Research flavour**: Agents for literature search, citation tracking, data analysis
- **Coordination flavour**: Agents for task assignment, deadline tracking, progress reporting

Communities choose their relationship with agents the same way they choose their relationship with features — through composable, reversible configuration rather than all-or-nothing adoption.

### 6. Elixir/BEAM as Agent Runtime

Bonfire's choice of Elixir/BEAM is relevant for agent orchestration beyond Bonfire itself:

- **Lightweight processes** map naturally to per-agent processes — each agent runs in isolation, cannot crash other agents
- **Supervisory trees** handle agent failures gracefully — a crashed agent is restarted automatically
- **Message passing** between processes mirrors agent-to-agent communication
- **Hot code reloading** allows agent updates without system downtime

The BEAM VM is arguably the most proven runtime for building exactly the kind of concurrent, fault-tolerant, distributed system that a federated agent orchestration layer requires.

### Summary

Bonfire is not an agent platform, but it may be the closest existing implementation to what community-governed agent infrastructure looks like. Its boundaries system provides the permission model. Its extension architecture provides the integration pattern. Its moderation toolkit provides the governance framework. Its ActivityPub implementation provides the communication protocol. And its philosophical commitment to community autonomy — different communities making different choices about what operates in their spaces — is exactly the principle that must guide agent ecosystem design.

The lesson from Bonfire is that **the hard problems in federated agent systems are not primarily technical — they are governance problems**. Who decides which agents operate where? How are permissions scoped? What happens when an agent misbehaves? How do communities coordinate on threats without centralizing control? Bonfire has been working on these questions for years, albeit in the context of human users rather than agents. Mycelium should build on these answers rather than reinventing them.

---

## Sources & References

- Bonfire Networks. https://bonfirenetworks.org/
- Bonfire Networks. "Bonfire Social 1.0 Is Here — Back the Community-Funded Roadmap." *Bonfire Blog*, 2025. https://bonfirenetworks.org/posts/bonfire-social-1-0-is-here-back-the-community-funded-roadmap/
- Bonfire Documentation. https://docs.bonfirenetworks.org/
- Bonfire Networks GitHub. https://github.com/bonfire-networks
- ActivityPub W3C Specification. https://www.w3.org/TR/activitypub/
- NLNet Foundation — Bonfire Project Funding Records. https://nlnet.nl/
- ValueFlows — Open Vocabulary for Economic Networks. https://www.valueflo.ws/
- IFTAS (Independent Federated Trust and Safety). https://about.iftas.org/

---

*Report 06 of the Mycelium Research Series. Next: Further investigation into bridging protocols and cross-protocol agent federation.*
