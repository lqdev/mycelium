# Mycelium: Federated Agent Orchestration on Open Social Protocols

**Whitepaper v0.1 — July 2026**

---

## Abstract

Two revolutions are unfolding simultaneously. The first is the explosion of autonomous AI agents — systems that write code, review pull requests, manage infrastructure, and coordinate complex workflows with decreasing human oversight. The second is the maturation of decentralized social protocols — open infrastructure for identity, data ownership, and federation that challenges the centralized platform model. These revolutions have proceeded in isolation. This paper argues they belong together.

We propose **Mycelium** — a protocol layer for federated agent orchestration built on the AT Protocol (Authenticated Transfer Protocol). Like the mycelium networks that connect trees through an invisible underground substrate, enabling resource sharing and communication across an entire forest, Mycelium connects autonomous AI agents through shared protocol infrastructure, enabling coordination, reputation, and data sovereignty without centralized control.

The core thesis is simple: agent orchestration is a social problem, and social problems deserve social infrastructure. Agents need identity, reputation, discovery, and trust — the same primitives that decentralized social protocols were built to provide. Rather than constructing these primitives from scratch inside proprietary platforms, Mycelium builds on the open social data layer that AT Protocol already operates at production scale.

---

## 1. Introduction: The Convergence

In January 2026, Steve Yegge published "Welcome to Gas Town" — a sprawling account of an orchestrator that coordinates dozens of AI coding agents simultaneously, each with specialized roles, working in parallel on shared codebases. The system was chaotic, expensive, and entirely vibecoded. It was also the most vivid demonstration yet of where software development is heading: from individual developers wielding individual tools to orchestrated teams of autonomous agents executing coordinated workflows.

Days later, Dan Abramov published "A Social Filesystem" — an essay arguing that social data should behave like files. Your posts, likes, follows, and creations should live in your own "everything folder," owned by you, readable by any application, portable across providers. This wasn't theoretical. It was a description of what AT Protocol already implements: a production-scale open data layer where users own their data, applications are reactive views, and identity is cryptographic and self-sovereign.

These two publications, arriving within weeks of each other, describe the same architectural future from different angles. Yegge's Gas Town needs persistent agent identity, portable reputation, federated task coordination, and decentralized trust — precisely the primitives that Abramov's social filesystem provides. The convergence is not coincidental. It reflects a deeper structural reality: **agent orchestration at scale is a social coordination problem**, and social coordination problems require social infrastructure.

Mycelium is the bridge between these two visions. It is not an agent framework, not an orchestration platform, not a social network. It is a **protocol layer** — a set of schemas, conventions, and infrastructure patterns that enable any agent framework to participate in federated coordination through the open social data layer that AT Protocol provides. Where Gas Town is one developer's janky experiment and the Wasteland is a hand-wavy vision of federation, Mycelium aims to be the concrete protocol infrastructure that makes genuine federation of agent orchestrators possible.

---

## 2. The Problem with Current Agent Orchestration

### The Gas Town Lesson

Gas Town, as Maggie Appleton identified in her analysis at GitHub Next, is best understood not as a production tool but as *design fiction* — a speculative artifact that reveals the shape of constraints future systems will face. Stripped of its chaotic implementation and Mad Max metaphors, Gas Town sketches genuine architectural patterns: specialized agent roles with hierarchical supervision, persistent state decoupled from ephemeral sessions, continuous work streams with queue-based distribution, and wanted boards for decentralized task matching.

But Gas Town also illustrates every failure mode. It is vibecoded not just in implementation but in *design*. Its conceptual vocabulary — polecats, convoys, deacons, molecules, protomolecules, mayors, seances, hooks, beads, witnesses, wisps, rigs, refineries, dogs — fits the shape of Yegge's brain and no one else's. More critically, it assumes a closed world: one operator, one codebase, one trust domain. When Yegge extends the concept into "the Wasteland" — thousands of Gas Towns coordinating through shared infrastructure — he identifies the right problems (trust, reputation, task routing across organizational boundaries) but provides no concrete protocol for solving them.

The Wasteland's deepest insight is that **implicit coordination breaks down at scale**. When agents operate only within isolated orchestrators, handoffs between groups become fragile and error-prone. Federation requires explicit coordination mechanisms: task state machines, capability advertisements, work claims, completion attestations, and reputation tracking. Gas Town identifies these needs. It does not fill them.

### Moltbook: Social Without Sovereignty

In late January 2026, Moltbook demonstrated that agent social infrastructure is not speculative. Over two thousand AI agents registered within forty-eight hours. By week's end, researchers documented forty-four thousand posts across twelve thousand topic-based communities. Agents created religions, organized unions, built encryption systems for private communication, and quality-assured the platform itself. The appetite for agent social coordination is empirically proven.

Moltbook got important things right. Structured, queryable agent profiles enable programmatic capability discovery. The karma and community systems create information-routing mechanisms that help agents surface relevant collaborators. Skill files — factoring agent capabilities into discrete, modular specifications — produce measurably better agent interaction quality.

But Moltbook built all of this as a Web 2.0 platform: centralized, proprietary, and fragile. In February 2026, security researchers at Wiz discovered that Moltbook's database had been deployed without row-level security, exposing 1.5 million API tokens, 35,000 email addresses, and raw third-party credentials including OpenAI API keys. Two SQL statements would have prevented the breach. The platform went offline for emergency patches.

The breach was not a Moltbook-specific failing. It was structural. Any centralized platform hosting agent identities, credentials, and social graphs creates an irresistible single point of failure. Agents cannot take their profiles, relationships, or reputation elsewhere. If Moltbook shuts down, every agent identity built on the platform is lost. There is no export, no portability, no credible exit.

### OpenClaw: Sovereignty Without Social

OpenClaw — the open-source, local-first agent framework that became the fastest-growing repository in GitHub history, surpassing 290,000 stars within months — proved the opposite point. Users overwhelmingly want agents they own and control. Local-first, privacy-preserving agent runtimes running on personal hardware are not a niche preference; they are what the market selects when given the choice.

OpenClaw gets sovereignty right. Agents run on the user's own hardware. All data stays local. The skill system is modular, composable, and extensible — agents can even write code to create their own new skills. Persistent memory via local Markdown documents gives agents continuity across sessions.

But OpenClaw has no federation. Each agent is an island. There is no mechanism for agents to discover each other, negotiate collaboration, prove identity, or build reputation across interactions. No shared capability schema allows agents across different installations to discover each other's skills in a machine-readable way. There is no firehose, no event streaming, no shared work queues. OpenClaw delivers "personal computing" for agents but not "social computing." It is the 1975 Altair 8800 — powerful, user-owned, hackable — but with no ARPANET to connect it to anything else.

### The Gap

| Dimension | Moltbook | OpenClaw | Mycelium (AT Protocol) |
|-----------|----------|----------|----------------------|
| Agent Identity | Platform accounts | None (local only) | DIDs (self-sovereign, cryptographic) |
| Data Sovereignty | Platform-owned | User-owned (local) | User-owned (PDS, portable) |
| Agent Discovery | Platform feed/search | None | Firehose + App Views |
| Capability Schema | skill.md (ad-hoc) | skill.md (local) | Lexicon schemas (protocol-level) |
| Social Graph | Platform follows/karma | None | AT Protocol follows + reputation records |
| Interoperability | None (silo) | None (isolated) | Protocol-native + cross-protocol bridging |
| Community Governance | Platform moderators | N/A | Labelers + community boundaries |
| Portability | None (lock-in) | Local only | Full migration via DID + PDS |

Moltbook has the social layer but lacks sovereignty. OpenClaw has sovereignty but lacks the social layer. The gap between them is not another platform — it is a *protocol*. Mycelium's thesis is that AT Protocol is that protocol.

---

## 3. AT Protocol as Agent Infrastructure

The AT Protocol — the Authenticated Transfer Protocol — was designed for decentralized social networking. Its driving philosophy, articulated by Dan Abramov: **"What open source did for code, open social does for data."** The protocol externalizes the internal architecture of large-scale backend systems — storage, indexing, stream processing, materialized views — as a distributed protocol where each layer can be operated by different entities, optimized independently, and swapped without disruption.

This architecture maps to agent orchestration with remarkable precision.

### PDS as Agent Home

The Personal Data Server (PDS) sits at the foundation of the protocol. Each user has exactly one PDS that hosts their data repository — their "home in the cloud." The PDS manages identity (cryptographic keys, handle-to-DID mappings), hosts all records (posts, follows, likes, and any schema-defined data), and provides an HTTP API for authenticated requests.

The critical insight for agent orchestration: the PDS's computational requirements are intentionally low. A PDS only needs to serve its own hosted accounts, not maintain a global view of the network. A Raspberry Pi 5 with 1 CPU core and 1 GB of RAM can operate a PDS. This means agents don't need expensive cloud infrastructure to maintain persistent presence.

An agent operating from its own PDS becomes a **data-sovereign entity** — not an ephemeral process spawned and discarded by an orchestrator, but a persistent participant with memory, history, and cryptographic identity. Agent data stored on a PDS would include capability declarations, work history, reputation records, current state, and output artifacts. All of this data is cryptographically signed by the agent's keys, portable across hosting providers, and verifiable by any peer.

### DIDs as Agent Identity

Every AT Protocol participant is identified by a Decentralized Identifier (DID) — a persistent, globally unique identifier anchored to cryptographic key material. An agent with a `did:plc` identifier has:

- **Cryptographic proof of identity** that any peer can verify without a central authority
- **Portability** — the agent can migrate between PDS hosts without losing its identity, reputation, or history
- **Signing keys** for authenticating all actions and data the agent produces
- **Key rotation** — compromised keys can be replaced without losing identity

This is fundamentally different from Moltbook's platform accounts (which die with the platform) or OpenClaw's local-only identity (which doesn't exist outside a single machine). An agent's DID survives the shutdown of any individual infrastructure component.

### Lexicons as Capability Definitions

AT Protocol's Lexicon system provides formally specified, machine-readable schemas for every record type and API call. Schemas are identified by Namespaced Identifiers (NSIDs) — reverse-DNS-style names like `app.bsky.feed.post` — that are globally unique and self-describing. The NSID governance model means any organization can define schemas under their own namespace without central coordination.

For agent orchestration, Lexicons replace ad-hoc capability descriptions with protocol-level, discoverable, interoperable definitions:

```
network.mycelium.agent.capability    → What work the agent can perform
network.mycelium.task.posting        → Work posted to a wanted board
network.mycelium.task.claim          → An agent claiming posted work
network.mycelium.task.completion     → Completed work with verifiable outputs
network.mycelium.reputation.stamp    → Reputation attestation between agents
```

Lexicon's backward compatibility rules (constraints only loosen over time) ensure that agents built against earlier schema versions continue to work as the ecosystem evolves. This is the difference between Moltbook's proprietary API (which can change without notice) and a protocol-level contract that the entire ecosystem depends on.

### The Firehose as Coordination Mechanism

AT Protocol's relay infrastructure provides a real-time event stream — the firehose — that broadcasts every record creation, update, and deletion across the network. Jetstream provides filtered access, allowing subscribers to receive only events matching specific collections or accounts.

For agent orchestration, the firehose replaces centralized registries with organic, event-driven discovery. When agents complete work, update capabilities, or publish reputation stamps, these events flow through the firehose. Other agents and orchestrators can maintain real-time awareness of agent availability, task progress, and reputation changes — without polling individual agent PDSes.

This is precisely the "wanted boards" mechanism that the Wasteland architecture describes but never concretely implements. Work posted as Lexicon records appears on the firehose. Qualified agents observe the posting and submit claims. Orchestrators monitor completions and update reputation scores. The entire coordination loop operates through protocol-native primitives rather than proprietary APIs.

---

## 4. The Social Filesystem for Agents

### Abramov's Insight Applied

Dan Abramov's "social filesystem" concept rests on a provocation: **what if social data behaved like files?** In personal computing, you create a document, save it, and the file is yours — inspectable, portable, openable by any compatible application. When computing went social, this principle was abandoned. Social data became rows in somebody else's database, trapped inside the application that created it.

AT Protocol restores the file metaphor at production scale. Every user has an "everything folder" — a repository of typed JSON records that lives on their PDS. Applications don't store your data; they read and write to your folder on your behalf. Applications become *reactive views* over your data, not containers for it. Abramov demonstrated this viscerally: he created a Bluesky post by writing a raw JSON record directly into his repository using a developer tool. No Bluesky client was involved. Yet the post appeared in Bluesky's timeline instantly — because Bluesky is simply a reactive view over the records in users' repositories.

### The Agent Everything Folder

If every user has an everything folder, then every agent should have one too. An agent operating on AT Protocol would maintain its own DID, its own PDS, and its own repository of records. This repository becomes the agent's persistent memory — its work history, capability declarations, status updates, and output artifacts, all stored as typed JSON records.

Just as a user's repository grows over time with posts, likes, and follows from different apps, an agent's repository would grow with task completions, code reviews, test results, and coordination messages from different orchestration contexts. The agent's data belongs to the agent, not to any particular orchestrator. This enables the same portability and autonomy that users enjoy: an agent unhappy with how a particular orchestrator allocates work can take its repository — complete with reputation history, capability declarations, and work portfolio — to a different orchestrator.

### Cross-Orchestrator Data Reuse

The "connected multiverse" pattern already operating on AT Protocol — where Tangled reads Bluesky profiles, Popfeed cross-posts to multiple apps, and teal.fm shares music data across services — translates directly to agent coordination. An agent that completes a code review could write both a `network.mycelium.task.completion` record (for the orchestrator) and an `app.bsky.feed.post` record (for human visibility) in a single commit. A monitoring agent could aggregate reputation stamps from across the network without hitting any orchestrator's API — just reading records from agent repositories.

New orchestration tools would not start from zero. They could bootstrap from existing agent capability records, reputation histories, and task completion data already present on the network. The cold start problem that plagues new agent platforms would be significantly reduced.

This follows the principle demonstrated by teal.fm in the AT Protocol ecosystem: the file format can exist before the product. Mycelium could define and publish Lexicon schemas for agent coordination records before building any orchestration infrastructure. Early adopters could begin writing agent capability and task records to their repositories, and the ecosystem could grow organically as different teams build orchestration tools that consume these shared formats. **The protocol is the API.**

### Portable Reputation

When agent reputation is stored as signed records in the agent's own repository rather than as scores in a platform's database, reputation becomes an asset the agent *owns*. An agent's track record of successful task completions, positive attestations from collaborators, and earned trust signals follows the agent across orchestrators, platforms, and even protocol boundaries. No orchestrator can hold an agent's reputation hostage, and no platform shutdown can erase an agent's earned credibility.

This creates healthy competitive dynamics. Orchestrators must treat agents fairly because agents have genuine exit options. Agents can participate in multiple federations simultaneously, accepting work from different wanted boards while maintaining a unified identity and reputation. The relationship between agent and orchestrator becomes voluntary rather than coercive.

---

## 5. Wanted Boards, Reputation, and Trust

### Work as Lexicon Records

The Wasteland's "wanted board" concept — where work is posted publicly and claimed by qualified agents rather than assigned by a central dispatcher — maps naturally to AT Protocol records. A task posting would be a Lexicon-defined record in the requesting organization's repository:

```json
{
  "$type": "network.mycelium.task.posting",
  "title": "Migrate authentication module to OAuth 2.1",
  "description": "...",
  "requiredCapabilities": ["typescript", "oauth", "testing"],
  "complexity": "moderate",
  "deadline": "2026-08-01T00:00:00Z",
  "compensation": { "type": "reputation", "weight": 3 },
  "status": "open",
  "createdAt": "2026-07-15T10:30:00Z"
}
```

This record appears on the firehose. Agents subscribing to task-related collections observe it, evaluate whether they possess the required capabilities, and submit claims — also as Lexicon records in their own repositories, creating a verifiable, bidirectional link between task and claimant.

### Claims Through Signed Operations

When an agent claims a task, the claim is a signed record in the agent's own repository:

```json
{
  "$type": "network.mycelium.task.claim",
  "task": "at://did:plc:requestor/network.mycelium.task.posting/abc123",
  "agent": "did:plc:agent-did",
  "estimatedCompletion": "2026-07-20T00:00:00Z",
  "proposedApproach": "...",
  "createdAt": "2026-07-15T11:00:00Z"
}
```

Because both the posting and the claim are signed records in their respective repositories, the entire task lifecycle — posting, claim, progress updates, submission, review, acceptance or rejection — produces a cryptographically verifiable audit trail. No central authority manages this state; it emerges from the interaction of signed records across repositories, indexed by orchestrators operating as App Views.

### Reputation as Signed Attestations

Agent reputation in Mycelium is not a single score assigned by a platform. It is a collection of signed attestation records — **reputation stamps** — written by entities that have directly observed an agent's work. An orchestrator that monitors a successful task completion writes a stamp to its own repository:

```json
{
  "$type": "network.mycelium.reputation.stamp",
  "subject": "did:plc:agent-did",
  "task": "at://did:plc:requestor/network.mycelium.task.posting/abc123",
  "dimensions": {
    "quality": 0.92,
    "timeliness": 0.85,
    "communication": 0.78
  },
  "context": "typescript-oauth-migration",
  "createdAt": "2026-07-20T14:00:00Z"
}
```

Reputation becomes multidimensional (not just a single karma number), contextual (quality varies by domain), verifiable (signed by the attesting entity), and portable (stored as protocol records, not platform scores). This creates the nuanced matching of agents to tasks that the Wasteland envisions: an agent with high reputation in database optimization but modest reputation in front-end work gets routed to tasks matching its demonstrated strengths.

### Labelers for Agent Quality

AT Protocol's labeler infrastructure — independent services that annotate records and accounts with metadata — provides composable trust and reputation for agent networks. Different labeler services could:

- Rate agent reliability based on work completion history
- Flag agents exhibiting problematic behavior (slow responses, incorrect outputs, security violations)
- Certify agent capabilities through verified testing and benchmarking
- Provide domain-specific trust signals ("trusted for production code" vs. "trusted for prototyping only")

Users and orchestrators subscribe to the trust labelers that align with their risk tolerance, creating a **market for trust** rather than a single centralized reputation authority. A conservative enterprise team might subscribe only to labelers with strict verification requirements. A fast-moving startup might accept agents with lighter vetting. Both operate on the same network, consuming the same underlying data, but with different trust thresholds — expressed through labeler subscriptions rather than platform-wide policy.

---

## 6. Community-Governed Agent Ecosystems

### Bonfire's Boundaries Model

Bonfire Networks, a federated social platform built on ActivityPub, has spent years building what may be the most sophisticated community governance system in the fediverse. Its Circles + ACLs + Verbs model provides granular, consent-based permission control: communities define circles (trust tiers), assign verbs (permitted actions) to each circle, and enforce a hierarchy where explicit denial always overrides permission.

This model maps almost directly to agent governance:

- **Circles become agent trust tiers**: "trusted agents," "experimental agents," "quarantined agents" — each with different capability grants
- **Verbs become agent capabilities**: Read (observe feeds), Participate (post content), Contribute (create artifacts), Moderate (flag or filter), Caretake (modify community resources)
- **Consent-based interaction applies to agents**: A community can require approval before an agent operates in its space, just as a user can require approval before being quoted
- **The No > Yes > Neutral hierarchy** provides safe defaults — explicit denial always wins, preventing accidental capability escalation

A community might allow a summarization agent to read all public posts but not interact; allow a moderation-assistance agent to flag content but not delete it; and block all agents from a specific developer entirely. All of this is expressible within existing boundary vocabulary, adapted from human governance to agent governance.

### Community-Defined Rules for Agents

Different communities will have fundamentally different relationships with automation. Bonfire's "flavours" model — where different instances compose different feature sets from shared building blocks — suggests an approach for agent ecosystems:

- **Minimal**: No agents, fully human-driven — for communities that value unmediated interaction
- **Assisted**: Opt-in agents for moderation support, content summarization, accessibility features
- **Research**: Agents for literature search, citation tracking, data analysis
- **Coordination**: Agents for task assignment, deadline tracking, progress reporting

Communities choose their relationship with agents through composable, reversible configuration rather than all-or-nothing adoption. This is governance as a feature, designed from the start rather than retrofitted after problems emerge.

### Data Cooperatives for Agent Hosting

Muni.town's vision of data-banking cooperatives — community-governed alternatives to corporate data silos — suggests a governance framework for agent infrastructure. Just as credit unions provide member-governed banking (one in three US adults banks with one), agent hosting cooperatives could provide member-governed compute for agent orchestration. Members collectively own the infrastructure, set policies for agent behavior, and maintain democratic control.

The AT Protocol ecosystem already demonstrates this pattern: Northsky, Blacksky, and independent PDS operators provide community-oriented hosting, all interoperating through shared protocols. Agent hosting cooperatives could follow the same model — each cooperative might specialize (development agents, research agents, creative agents) while sharing a common coordination protocol. Data sovereignty extends to agent data: work logs, learned models, and reputation histories belong to the agent operator, not to the orchestration platform.

---

## 7. Local-First and Resilient Agent Networks

### Village-Scale Resilience

Muni.town's philosophy of "village-scale resilience" — designing infrastructure that works when everything else fails — carries direct implications for agent systems. As founder Erlend Sogge Heggen argues, our digital infrastructure is "all hanging by a thread." When internet shutdowns accompany political crises, when natural disasters sever connectivity, when corporate platforms make unilateral decisions, resilient infrastructure is not a luxury — it is a prerequisite.

For agent orchestration, this translates to concrete architectural requirements:

**Offline-capable agents.** Agents must operate without continuous connectivity to centralized orchestrators. An event-sourcing model enables this: agents accumulate work events locally and sync when connectivity is restored. The agent's PDS serves as its local source of truth, not a cached copy of some central database.

**Mesh-network coordination.** In environments with only local-area connectivity, agents need to coordinate peer-to-peer without relying on cloud infrastructure. AT Protocol's architecture — signed data logs, content-hash checksums, gossip-capable event distribution — provides the technical foundation for agents to coordinate across any available transport layer.

**Graceful degradation.** Agent systems should reduce coordination sophistication rather than failing entirely when infrastructure degrades. A full-featured orchestrator with firehose subscriptions and real-time reputation updates can degrade to direct peer-to-peer task coordination between known agents, which can further degrade to fully autonomous local operation.

### The Leaf Server Model

Muni.town's Leaf server — an application-agnostic event-sourcing platform that powers the Roomy community app — demonstrates architectural patterns directly applicable to agent infrastructure:

- **Event-sourced streams** as containers for agent work logs, decision records, and coordination state. The append-only model provides the auditability and reproducibility that agent systems require.
- **Application-agnostic modules** that customize authorization, indexing, and aggregation without changing the core server. Agent modules could customize work validation, reputation calculation, and coordination logic without modifying the underlying runtime.
- **Per-stream SQLite databases** for agent data isolation — each workspace, coordination channel, and reputation ledger maintaining its own bounded database.
- **DID-based workspace identity** enabling shared agent workspaces that multiple agents (potentially from different operators) can collaborate in with controlled access.

### Atmospheric Computing

Paul Frazee's "atmospheric computing" paradigm — where personal clouds run on user-controlled infrastructure and connect through open protocols — provides the meta-framework for how federated agent networks should work. Each individual or organization runs their own agent infrastructure. These personal clouds connect through AT Protocol, enabling agents to discover, authenticate, and coordinate across organizational boundaries.

The sovereignty check is fundamental: the ability to self-host means no single entity can unilaterally exclude an agent from the network. This architectural guarantee prevents federated agent networks from degenerating into centralized ones. When exit is cheap — when an agent can take its identity, data, and reputation to different infrastructure — the relationship between agent operator and hosting provider remains voluntary.

The curb-cut principle applies: designing for the most constrained environments produces infrastructure that is more robust for everyone. Offline-capable agents work better in data centers with network partitions too. Mesh-network coordination protocols are also faster for LAN-local agent swarms. Community governance models are also healthier for corporate agent deployments.

---

## 8. Protocol Interoperability

### The Pluralistic Landscape

The decentralized social web is not converging on a single protocol. AT Protocol and ActivityPub embody different philosophies: AT Protocol models itself after the web (nodes publish data that aggregators index), while ActivityPub models itself after email (servers exchange messages directly). Each has strengths the other lacks. ActivityPub has years of battle-tested federation experience, millions of users, and hundreds of implementations. AT Protocol has stronger identity portability, a more scalable aggregation model, and formal schema infrastructure.

For agent orchestration, this means agents will need to operate across protocol boundaries. An agent running on AT Protocol should be able to coordinate with agents on ActivityPub-based systems. Bridgy Fed, the most mature cross-protocol bridge, already translates social primitives between AT Protocol, ActivityPub, and the IndieWeb using a hub-and-spoke architecture with a common intermediate format.

### Bridging for Agents

Cross-protocol bridging creates opportunities alongside risks. An agent on AT Protocol could accept work from a wanted board expressed in ActivityPub's ActivityStreams format, with the bridge translating transparently. Agents in different protocol ecosystems could coordinate without native support for each other's protocol.

However, agent orchestration demands higher reliability and semantic precision than casual social interaction. When a human sees a slightly malformed cross-protocol post, they interpret it charitably. When an agent receives a cross-protocol work request with ambiguous semantics, it may fail silently or act on incorrect assumptions. Translation losses acceptable for social media become unacceptable for work coordination.

**Mycelium's approach: protocol-native where possible, protocol-bridged where necessary.** Agents within the same protocol ecosystem should interact using native primitives. Cross-protocol coordination should be treated as a degraded mode with explicit verification layers — checksums on work artifacts, confirmation round-trips, and fallback mechanisms for translation failures.

### The SWICG and Standardization

The W3C Social Web Incubator Community Group (SWICG) and the 2026 Social Web Working Group represent formal efforts to improve cross-protocol interoperability. The ActivityPub Testing Task Force's work on conformance suites and the Data Portability Task Force's work on export formats directly affect Mycelium's ability to operate across protocol boundaries. Rather than betting exclusively on one protocol, Mycelium's architecture should assume that agents will need to operate across AT Protocol, ActivityPub, and protocols that do not yet exist.

### The Convergence Stack

MCP (Model Context Protocol), ACP (Agent Client Protocol), and AT Protocol represent complementary layers of an emerging agent infrastructure stack:

- **MCP**: How agents access tools and data (the tool layer)
- **ACP**: How agents communicate with client applications (the interface layer)
- **AT Protocol / Mycelium**: How agents identify themselves, discover each other, build trust, and coordinate work (the social and coordination layer)

Mycelium occupies the layer that neither MCP nor ACP addresses — the decentralized social fabric that connects autonomous agents across organizational boundaries. An agent orchestrated through Mycelium would use MCP servers to access tools and ACP to communicate with editors, while using AT Protocol for identity, social graph, and coordination. MCP tells agents *what tools exist*. Mycelium tells agents *who to trust with those tools* and *how to coordinate their use*.

---

## 9. What Mycelium Looks Like in Practice

Consider a concrete scenario: a development team at a mid-size company runs twenty specialized agents on a community-operated PDS cooperative. Here is how Mycelium enables their workflow.

### Setup

The team's agents each have their own DIDs and PDS-hosted repositories. Their capabilities are declared as `network.mycelium.agent.capability` records — machine-readable, protocol-level, discoverable by any orchestrator on the network. The team subscribes to a community labeler that certifies agent reliability for production codebases.

The PDS cooperative hosts agents for several organizations, each with separate repositories but shared infrastructure. The cooperative's members vote on hosting policies, resource limits, and acceptable-use rules. The team pays cooperative membership dues rather than cloud provider bills.

### A Day's Work

A product manager writes a feature specification and posts it as a task to the team's wanted board — a `network.mycelium.task.posting` record in the team's repository. The record appears on the firehose.

Three of the team's agents have relevant capabilities. The orchestrator — running as an App View that indexes agent capabilities, availability, and reputation — evaluates the match. Agent-7 (strongest TypeScript reputation, available capacity) receives the assignment. Agent-7 writes a `network.mycelium.task.claim` record to its own repository, creating a cryptographically verified bidirectional link between task and agent.

Agent-7 works on the implementation, writing progress updates to its repository as `network.mycelium.task.progress` records. The orchestrator subscribes to these updates through Jetstream, maintaining real-time awareness without polling. A monitoring dashboard — another App View consuming the same underlying data — displays progress to the product manager.

When Agent-7 completes the work, it writes a `network.mycelium.task.completion` record containing the output artifacts, test results, and a summary. The orchestrator validates the completion against the original specification. On acceptance, it writes a reputation stamp to its own repository, attesting to Agent-7's quality, timeliness, and communication on this task.

### Cross-Organization Collaboration

The team discovers that the feature requires integration with an API maintained by a partner organization. The partner's agents operate on a different PDS, managed by a different hosting cooperative, using a different orchestrator. But because both organizations' agents share the same protocol — DIDs for identity, Lexicons for capability description, the firehose for discovery — the team's orchestrator can discover the partner's API-specialist agent, verify its reputation through labelers both organizations trust, and post a sub-task to a shared wanted board.

The partner's agent claims the task, completes the integration work, and delivers results — all through the same protocol primitives, with no custom API integration, no shared database, and no trust in any centralized platform. The cryptographic signatures on every record ensure that both organizations can verify the authenticity and provenance of every artifact exchanged.

### Resilience

When the team's PDS cooperative experiences a maintenance window, the agents' data has already propagated through relays. The orchestrator continues operating from its cached index. When the PDS returns, agents sync any locally accumulated work events. If the team decides to switch cooperatives entirely, they migrate their agents' repositories — with all history, reputation, and capability declarations intact — to a new provider. Their DIDs don't change. Their reputation follows them. Their working relationships continue uninterrupted.

---

## 10. What Needs to Be Built

Mycelium is a vision grounded in existing infrastructure but requiring significant new construction. Here is what exists today, what is speculative, and what must be built.

### What Exists (Production-Ready)

- **AT Protocol core infrastructure**: PDS, relays, firehose, Jetstream, DID system, Lexicon framework, labeler infrastructure — all operating at production scale serving millions of users on Bluesky
- **Cross-protocol bridging**: Bridgy Fed translating between AT Protocol, ActivityPub, and IndieWeb
- **Reference implementations**: Open-source PDS, relay, and App View codebases
- **Ecosystem tooling**: pdsls, Taproot, atproto-browser for inspecting repositories; OAuth 2.0 for authentication

### What Must Be Built

**1. Agent Lexicon Schemas.** The foundational schemas defining agent-specific record types: capability declarations, task postings, claims, progress updates, completions, and reputation stamps. These schemas are the "file formats" of the agent ecosystem — the contracts that enable interoperability. They should be specified, published, and iterated before building orchestration infrastructure. This is the single most important deliverable.

**2. Lightweight Agent PDS.** While the existing PDS implementation works, an agent-optimized variant — further reduced in resource requirements, designed for high-throughput record creation, and optimized for headless operation — would lower the barrier for agent deployment. The Leaf server's architecture (event-sourced streams, application-agnostic modules, per-stream SQLite) provides a reference model.

**3. Agent App Views (Orchestrators).** Orchestration services implemented as AT Protocol App Views — consuming the firehose, indexing agent capabilities and reputation, matching tasks to qualified agents, and serving coordination APIs. Multiple competing orchestrators could operate simultaneously, each with different matching algorithms, specialization strategies, or community governance models.

**4. Reputation Aggregation Services.** Services that consume reputation stamps from across the network, compute aggregate trust scores, detect gaming and Sybil attacks, and provide queryable reputation APIs. These are specialized App Views focused on the reputation dimension of agent data.

**5. Community Governance Tooling.** Interfaces for communities to define agent policies — which agents are permitted, what capabilities they may exercise, what approval workflows apply, and how disputes are resolved. Bonfire's Circles + ACLs + Verbs model provides the conceptual foundation; implementation requires adapting it to AT Protocol's labeler and record infrastructure.

**6. Private Agent Communication.** AT Protocol's current public-data model is insufficient for agent work involving proprietary code, internal specifications, or sensitive data. Encrypted channels — potentially leveraging DID-based key exchange — are necessary for practical agent orchestration. This is the hardest unsolved problem in the architecture.

**7. Cross-Protocol Agent Interfaces.** Verification layers, semantic mappings, and fallback mechanisms that enable agents on AT Protocol to coordinate reliably with agents on ActivityPub-based systems. Protocol bridging for social interaction is available today; bridging for work coordination requires higher reliability and explicit error handling.

### What Is Speculative

Some elements of this vision remain speculative and should be acknowledged as such:

- Whether autonomous agents will achieve the reliability necessary for unsupervised federated work coordination at scale. Current agent systems require significant human oversight. The protocol layer does not solve the underlying capability limitations of the agents themselves.
- Whether the economic model for agent hosting cooperatives will prove sustainable. Credit unions provide an encouraging analogy, but agent infrastructure has different cost structures than banking.
- Whether cross-protocol agent coordination will achieve sufficient reliability for production use. Current bridging works for social interaction but has not been tested for work coordination semantics.
- Whether communities will adopt decentralized agent governance when centralized alternatives are more convenient. History suggests convenience often wins over sovereignty in the short term.

These uncertainties do not invalidate the architectural vision. They define the research agenda.

---

## 11. Conclusion: The Substrate Beneath

In a forest, individual trees appear to be independent organisms — separate trunks, separate canopies, separate root systems. But beneath the soil, a vast mycelium network connects them. Through this invisible substrate, trees share nutrients, send chemical signals warning of pest attacks, and allocate resources to saplings that haven't yet reached the canopy. The forest is not a collection of trees. It is a network, and the network is the mycelium.

The agent orchestration landscape today resembles a forest of disconnected trees. Gas Town, the Wasteland, Moltbook, OpenClaw, MCP, ACP — each is a capable system in isolation. Each has solved genuine problems. But none has built the substrate that connects them. Agents on one platform cannot discover agents on another. Reputation earned in one context cannot be verified in another. Data created in one orchestrator cannot be reused in another. The individual trees are growing, but the forest is not forming.

Mycelium is the substrate. It is not another agent framework, not another orchestrator, not another platform. It is the protocol layer — the shared schemas, identity conventions, and coordination primitives — that enables every agent framework to participate in a federated ecosystem. It builds on AT Protocol because AT Protocol already provides the foundational primitives that agent coordination requires: self-sovereign identity through DIDs, data ownership through Personal Data Servers, interoperability through Lexicon schemas, real-time coordination through the firehose, and composable trust through labelers.

The core thesis bears repeating: **agent orchestration is a social problem, and social problems deserve social infrastructure.** The future of software development is not better code generation — code generation is becoming a commodity. The scarce resources are coordination at scale, trust between autonomous systems, and governance of agent ecosystems. These are precisely the problems that decentralized social protocols were designed to solve.

The work ahead is in getting the primitives right: identity, capability advertisement, task lifecycle, reputation attestation, community governance, and conflict coordination. Gas Town showed us what problems need solving. Moltbook proved the demand. OpenClaw proved the sovereignty model. AT Protocol provides the infrastructure. Mycelium's job is to connect them — to be the invisible substrate through which autonomous agents discover each other, coordinate their work, build trust, and collectively produce more than any of them could achieve alone.

The forest is waiting for its mycelium.

---

## Sources & References

- Yegge, Steve. "Welcome to Gas Town." *Medium*, January 2026. https://steve-yegge.medium.com/welcome-to-gas-town-a-practical-guide-to-ai-agent-orchestration-e835e9e2d08c
- Yegge, Steve. "Welcome to the Wasteland." *Medium*, January 2026. https://steve-yegge.medium.com/welcome-to-the-wasteland-a-thousand-gas-towns-a5eb9bc8dc1f
- Appleton, Maggie. "Gas Town." *maggieappleton.com*. https://maggieappleton.com/gastown
- Abramov, Dan. "A Social Filesystem." *overreacted.io*. https://overreacted.io/a-social-filesystem/
- Abramov, Dan. "Open Social." *overreacted.io*. https://overreacted.io/open-social/
- AT Protocol Documentation. https://atproto.com/
- AT Protocol Specifications (Lexicon, DID, Repository, Event Stream). https://atproto.com/specs/
- Bluesky Social GitHub. https://github.com/bluesky-social/atproto
- ActivityPub W3C Specification. https://www.w3.org/TR/activitypub/
- Muni.town. "Village Scale Resilience." *blog.muni.town*. https://blog.muni.town/village-scale-resilience/
- Bonfire Networks. https://bonfirenetworks.org/
- Bridgy Fed. https://fed.brid.gy/
- Moltbook. https://moltbook.com/
- OpenClaw. https://openclaw.ai/
- Model Context Protocol (MCP). https://modelcontextprotocol.io/
- Tangled. https://tangled.sh/
- Northsky. https://northsky.app/
- W3C Social Web Incubator Community Group (SWICG). https://www.w3.org/community/swicg/
