# Toward a Pluralistic Social Computing Stack

> **Mycelium Whitepaper 03**
> A grand synthesis: how decentralized identity, user-sovereign storage, schema interoperability, federated streaming, open social applications, AI agent orchestration, and community governance are converging into a new computing stack — one that nobody designed, but that we can intentionally shape.

---

## 1. Introduction: The Stack We're Building

No single entity designed this stack. No committee chartered it. No corporation planned it. And yet, if you look at what is being built across a dozen independent projects — AT Protocol's decentralized identity and data sovereignty, ActivityPub's battle-tested federation, Bonfire's community governance primitives, Muni Town's local-first resilience, Gas Town's chaotic agent orchestration, OpenClaw's sovereign agent runtimes, Moltbook's proof that agents crave social infrastructure, Bridgy Fed's cross-protocol bridging — a coherent architecture is emerging. Not coherent in the sense of being polished or complete, but coherent in the way that the early internet was coherent: independent layers solving independent problems, gradually discovering that they compose.

The claim of this whitepaper is simple and ambitious: **we are witnessing the emergence of a pluralistic social computing stack** — an architecture in which humans and AI agents share the same infrastructure for identity, data, communication, and governance. Not because someone decided they should, but because the problems each layer solves are the same problems whether the participant is a person posting on Bluesky or an agent claiming a task from a wanted board.

This is not a prediction. The pieces exist. Bluesky has 20+ million users on AT Protocol. The Fediverse has millions more on ActivityPub. OpenClaw became the fastest-growing open-source repository in GitHub history. Moltbook registered thousands of agents in its first 48 hours. Bridgy Fed bridges protocols that were never designed to interoperate. The question is no longer whether these pieces can coexist. The question is whether we can compose them intentionally — filling the gaps, building the bridges, and establishing the governance — before the window closes and centralized alternatives calcify.

This whitepaper maps the stack as it exists today, assesses what works, identifies what is missing, and proposes a path forward. It is both manifesto and technical roadmap. It is honest about uncertainty. And it is grounded in specific mechanisms, specific projects, and specific architectural decisions — not in abstractions.

---

## 2. Layer 0: Identity

Everything starts with identity. Before an entity — human or agent — can store data, publish schemas, federate messages, run applications, or participate in governance, it must be able to say *who it is* in a way that others can verify, that survives infrastructure changes, and that no single authority can revoke.

Three identity systems are in production today, each embodying different tradeoffs.

### AT Protocol: Decentralized Identifiers (DIDs)

AT Protocol supports two DID methods. **`did:plc`** is self-authenticating: the identifier encodes cryptographic material, a PLC directory maintains an audit log of DID document updates, and rotation keys allow identity changes independent of any hosting provider. An agent or user with a `did:plc` can rotate compromised keys, migrate to new infrastructure, and recover from PDS failure — all without cooperation from anyone. **`did:web`** ties identity to a domain the operator owns, resolved via standard HTTPS. It is simpler to deploy but fragile: domain seizure means identity loss.

Both methods publish DID documents containing a signing key (authenticating all data the entity produces), a PDS service endpoint (current hosting location), and human-readable handles resolved via DNS. Handles can change freely — the protocol references the permanent DID, not the mutable name.

The critical property: **identity is independent of infrastructure**. A user can migrate between PDS providers. An agent can switch orchestrators. Neither loses their identity, their social graph, or their history. This is not theoretical — tools like pdsmoover.com enable full repository migration today.

### ActivityPub: HTTP URIs

ActivityPub identity is an HTTP URI controlled by the user's home server — `https://mastodon.social/users/alice`. The actor exposes inbox and outbox endpoints, and other servers discover them via WebFinger. This is simple, leverages existing web infrastructure, and has proven itself across millions of users and hundreds of implementations over eight years.

The weakness is structural: **identity is inseparable from server infrastructure**. If the server dies, the identity becomes inaccessible. Followers cannot follow a migrated account. Social graph fragments. FEP-7628 (Move Actor) provides a best-effort redirect, but only works if the old server remains accessible — useless in the most common failure case. Hubzilla's Zot protocol pioneered nomadic identity (cloning identity across multiple servers, using private keys as the root of identity), but cognitive overhead limited adoption.

### Agent Identity: The Uncharted Territory

Neither system was designed for agents, but both can accommodate them. An agent with a `did:plc` becomes a first-class participant in AT Protocol — with cryptographic proof of identity, portable reputation, and signing keys authenticating every action. An agent as an ActivityPub actor receives an inbox and outbox, can post, follow, and be followed. Moltbook proved explosive demand for agent identity (2,000+ agents, 10,000+ posts in 48 hours) but built it on platform accounts — and a single breach exposed 1.5 million API tokens. OpenClaw proved agents can be sovereign but gave them no networked identity at all.

### Assessment

AT Protocol's DID system is the most mature foundation for identity in this stack. It solves the portability problem that has plagued ActivityPub for years. It provides cryptographic verification without central authorities. And it works today — not as a specification in progress, but as production infrastructure serving millions of users. The limitation is real: DIDs cannot be changed once created (no migration from `did:plc` to `did:web`), and the PLC directory is currently operated by Bluesky. But these are operational constraints, not architectural ones. The identity model itself is sound.

For agents specifically, the DID model provides something no alternative offers: **an identity that belongs to the agent and its operator, not to whatever platform or orchestrator happens to be running it today**. This is the foundation on which everything else — memory, reputation, coordination, governance — is built.

---

## 3. Layer 1: Personal Data Storage

With identity established, the next question is: where does data live, and who controls it?

### AT Protocol: Personal Data Servers (PDS)

The PDS is the foundation of the AT Protocol architecture. Each user (or agent) has a repository of typed JSON records, organized into collections by Lexicon NSID, encoded as CBOR, and structured as a Merkle Search Tree. Every write produces a signed commit containing the new root hash. The entire repository state reduces to a single cryptographic commitment that anyone can verify.

Dan Abramov's "everything folder" metaphor captures the paradigm shift: your repository contains all your data — posts, likes, follows, recipes, code reviews, agent capability declarations — organized into namespaced collections. Apps are projections over this data, not containers for it. Delete a record from your folder and it disappears from every app. Create a record and every subscribed app reacts. The app's database is a derived cache, not the source of truth.

This is not theoretical. Abramov demonstrated it live: creating a Bluesky post by writing a raw JSON record to a repository using `pdsls`. No Bluesky client involved. The post appeared in the timeline because Bluesky is a reactive view over repository records.

The PDS is lightweight — demonstrated to run on a Raspberry Pi 5 with one CPU core and one GB of RAM. It only serves its hosted accounts; it maintains no global view. This makes it feasible for individuals, small organizations, and — critically — for agents to operate their own data stores.

### Solid Pods

Tim Berners-Lee's Solid project shares the vision of user-controlled data pods, but takes a different architectural path: RDF and Linked Data for schemas, SPARQL for queries, WebID for identity, and fine-grained access control (WAC/ACP) for permissions. Solid is more general-purpose and more standards-oriented; AT Protocol is more opinionated and more social-network-focused. Solid's emphasis on fine-grained access control addresses the private data problem that AT Protocol's public-data-only model currently leaves unsolved. But Solid has struggled with adoption — the standards-first approach produced academic rigor without the compelling application that drives developer adoption.

### Leaf Server: Event-Sourced Streams

Muni Town's Leaf server offers a complementary model: application-agnostic, event-sourced data streams with per-stream SQLite databases. Unlike PDS repositories (which are individual), Leaf streams are often communal — representing shared spaces with controlled multi-user read/write access. Events are minimal (index, user DID, binary payload), format-agnostic, and append-only. Modules customize authorization, materialization, and aggregation using SQL, without modifying the core server.

Leaf exists because PDS has real limitations for certain use cases: private data (PDS records are public), high-frequency events (PDS has rate limits), and local-first sync (canonical AT Protocol source makes offline operation harder). Leaf streams use DIDs published to plc.directory and support rotation keys, maintaining sovereignty while solving problems the PDS was not designed for.

### OpenClaw: Local-First Agent Storage

OpenClaw stores agent state locally — persistent memory via Markdown documents, skill files in TypeScript or Python, and cron-based proactive operations. The agent's data lives on the user's hardware: Mac, Windows, Linux, even a Raspberry Pi. This is genuine sovereignty — no cloud dependency, no platform risk. But it is also isolation: no federation, no shared identity, no mechanism for other agents to discover or verify anything about this agent.

### Assessment

The PDS is the most production-proven personal data store in this stack. It combines cryptographic verification, portability, lightweight operation, and a growing ecosystem of applications reading and writing shared data. Leaf fills a genuine gap for private, high-frequency, and communal data. OpenClaw proves that powerful agent capabilities can run on user hardware, but needs a network layer. Solid provides the most sophisticated access control model but has not achieved comparable adoption.

The strategic synthesis: **PDS for public, portable, verifiable data. Leaf for private, real-time, communal data. Local storage for sovereign, offline-capable agent operations.** These are not competing alternatives — they are complementary layers serving different requirements within the same stack.

---

## 4. Layer 2: Schemas & Formats

Data without shared meaning is noise. The stack requires a schema layer that defines what records look like, what APIs accept and return, and how different applications interoperate without prior coordination.

### AT Protocol Lexicons

Lexicons are AT Protocol's schema language — the "file formats" for social data. Every Lexicon is assigned a Namespaced Identifier (NSID) using reverse-DNS notation: `app.bsky.feed.post`, `pub.leaflet.publication`, `sh.tangled.repo`, `fm.teal.alpha.feed.play`. Own the domain, own the namespace — publish schemas without central permission.

Lexicons define four types: **records** (data stored in repositories), **queries** (HTTP GET operations), **procedures** (HTTP POST operations), and **subscriptions** (WebSocket event streams). They specify not just types but constraints: a field is not just a `string` but "a string of at most 300 Unicode graphemes." They support protocol-specific types: `at://` URIs linking to other records, CID links for content-addressed references, blob references for binary data.

The backward compatibility rules are critical: **once published, constraints can only be loosened, never tightened**. Incompatible changes must be published under a new NSID. This ensures older implementations always validate data from newer implementations — a property that agent ecosystems, with their potentially long-lived and hard-to-update participants, desperately need.

Cross-app data reuse is already happening in production. Tangled prefills avatars from Bluesky profiles. Popfeed cross-posts to both Bluesky and Leaflet. Anisota renders Leaflet publications. Blento displays teal.fm scrobbles — even though the teal.fm product did not yet exist when users began writing scrobble records. The Lexicon definition existed before the product. Data accumulated before the official app. **File formats outlive and predate the applications that use them.**

### Activity Streams and JSON-LD

ActivityPub uses Activity Streams 2.0 with JSON-LD for extensibility. The `@context` mechanism maps property names to globally unique namespace URIs, theoretically enabling unlimited extension without namespace collisions. In practice, many implementations treat payloads as flat JSON with conventions, ignoring the JSON-LD machinery. The result is a two-tier ecosystem with subtle interoperability bugs: extensions like emoji reactions, quote posts, and post editing exist in multiple incompatible formats across different platforms.

Activity Streams is conceptually elegant. Its actor-activity-object model naturally represents social interactions. Its extensibility has enabled remarkable diversity: Mastodon's microblogging, PeerTube's video sharing, Lemmy's link aggregation, Bonfire's economic coordination via ValueFlows. But elegance comes at a cost. The spec defines data formats but leaves processing semantics ambiguous. Different implementations make different edge case choices. Interoperability is aspirational rather than guaranteed.

### Assessment

Lexicons win on developer experience. The reverse-DNS naming prevents collisions without central coordination. The backward compatibility rules prevent ecosystem fragmentation. The constraint that schemas are "the API" — Lexicon is the protocol — eliminates the gap between data format and application behavior that plagues ActivityPub. The trade-off is that Lexicons are more opinionated and less theoretically flexible than JSON-LD. But in practice, that opinionation produces interoperability, and interoperability is what a multi-application, multi-agent ecosystem requires.

For agent ecosystems specifically, Lexicons provide what `skill.md` files and ad-hoc capability descriptions cannot: **formally specified, machine-readable, discoverable, versioned capability declarations** that agents across different implementations can read, understand, and act upon. When an agent publishes a `mycelium.agent.capability` record conforming to a shared Lexicon, any orchestrator anywhere in the network can parse it without custom integration.

---

## 5. Layer 3: Federation & Streaming

Data stored in personal repositories needs to flow. The stack requires mechanisms for distributing updates, discovering content, and enabling real-time coordination across independent infrastructure.

### AT Protocol: Firehose, Relays, and Jetstream

AT Protocol's distribution model inverts traditional federation. Rather than servers pushing content to each other peer-to-peer, relays aggregate repository updates into a single unified stream — the **Firehose**. App Views, feed generators, labelers, and any custom consumer subscribe to this stream, index what they need, and build their own materialized views.

The Firehose is CBOR-encoded, transported over WebSocket, and cryptographically verifiable: every record can be validated against the creator's public keys in their DID document, regardless of which relay transmitted it. **Jetstream** provides a developer-friendly alternative: JSON over WebSocket with optional filtering by collection or account. The trade-off is that Jetstream events lack cryptographic signatures — consumers must trust the Jetstream operator.

**Rainbow** provides fan-out, maintaining a single upstream relay connection and re-broadcasting to many clients. The protocol anticipates eventual sharding by account identifiers. Throughput currently handles hundreds to thousands of events per second.

The architectural insight is profound: the internal components of a centralized platform — caches, replicas, stream processors, materialized views — are externalized as the protocol itself, distributed across independent operators. Every App View's database is explicitly a cache, rebuildable from firehose replay. As Abramov put it: "I could delete those tables in production, then use Tap to backfill the database from scratch. I'm just caching a slice of global data."

### ActivityPub: Server-to-Server Federation

ActivityPub's distribution model is peer delivery. When an actor publishes an activity, their server identifies recipients, retrieves actor profiles via HTTP GET to extract inbox endpoints, and delivers via authenticated HTTP POST. Authentication uses HTTP Signatures — cryptographic proof the server holds the private key for the claimed actor.

This model has proven itself across millions of users and billions of federated objects over eight years. It prioritizes server autonomy: each instance maintains its own policies, its own moderation decisions, its own federation relationships. The result is genuine decentralization — no single entity controls the network.

The weakness is fanout. A popular account with thousands of followers distributed across thousands of servers requires thousands of individual HTTP POSTs per publication. The sending server bears the full computational cost. There is no broadcast primitive, no shared delivery infrastructure, no protocol-mandated retry strategy. AT Protocol's relay model centralizes distribution cost but solves fanout elegantly; ActivityPub distributes governance power but creates scaling bottlenecks.

### Bridgy Fed: Protocol Bridging

Bridgy Fed demonstrates that protocol boundaries are not walls. Using an intermediate format approach (ActivityStreams 1.0 as a hub-and-spoke converter), it translates between AT Protocol, ActivityPub, and IndieWeb standards — avoiding the combinatorial explosion of pairwise translators. Bluesky users can follow Mastodon users; both can interact with independent website publishers.

But bridging is lossy. Identity models are fundamentally incompatible: DIDs lose self-sovereignty when bridged to ActivityPub; ActivityPub identities cannot generate the cryptographic proofs AT Protocol consumers expect. Semantic mapping is imperfect: an `app.bsky.feed.like` and an ActivityPub `Like` activity carry subtly different semantics. Privacy contexts clash: a post on a small Mastodon instance bridged to Bluesky's App View expands its effective audience by orders of magnitude.

### Assessment

Both federation models are needed. AT Protocol's relay/firehose model provides the global aggregation and real-time streaming that agent discovery and coordination require. ActivityPub's peer delivery model provides the server autonomy and governance independence that community self-determination requires. Bridgy Fed proves they can coexist. The honest assessment: **no complete solution to cross-protocol interoperability exists today**, but the existing bridges are functional enough to build on.

For agent orchestration specifically, the Firehose is transformative. When agents complete work, update capabilities, or publish reputation stamps, these events flow through the stream. Other agents and orchestrators maintain real-time awareness without polling individual repositories. This replaces centralized agent registries with organic, event-driven discovery — precisely the "wanted boards" model that Gas Town's Wasteland architecture envisioned but never implemented at the protocol level.

---

## 6. Layer 4: Applications & Agents

The layers below — identity, storage, schemas, federation — exist to support what runs on top: applications used by humans, applications used by agents, and increasingly, applications used by both.

### Human Applications

The AT Protocol ecosystem (the "Atmosphere") already demonstrates the everything-ecosystem vision. **Bluesky** provides microblogging. **Tangled** provides Git hosting and code collaboration. **Leaflet** provides long-form publishing. **Semble** and **Smoke Signal** provide event management. **WhiteWind** provides blogging. **teal.fm** is accumulating music scrobbles from third-party tools before its official product even launches. Each application reads and writes to the same user repositories. Data created in one context is available in another. No API integrations, no data export/import, no separate accounts.

In the ActivityPub ecosystem, **Mastodon** remains the flagship microblogging platform. **PeerTube** handles federated video. **Pixelfed** does Instagram-like photo sharing. **Lemmy** provides Reddit-like link aggregation. **Bonfire** — perhaps the most architecturally ambitious — provides a modular, extensible framework where communities assemble purpose-specific applications from composable extensions. Its flavours model lets the same codebase serve as a social network, a coordination tool, an open science platform, or a community governance space.

### Agent Applications

**Gas Town** (Steve Yegge's agent orchestrator) demonstrated that managing dozens of parallel coding agents is achievable, with specialized roles (Mayor, Polecats, Witness, Refinery, Deacon, Dogs), persistent task state via "Beads" stored in Git, and queue-based work distribution. Its conceptual maturity is high — the patterns of hierarchical supervision, persistent state separate from ephemeral sessions, and explicit coordination mechanisms are sound. Its implementation is a research artifact: 75,000 lines of vibecoded code, expensive to run, requiring constant human steering.

**The Wasteland**, Gas Town's federation extension, introduced wanted boards (decentralized work posting), state-based task progression, and reputation systems with multidimensional stamps — "character sheets" for agents tracking success rates by task type. These are exactly the primitives a federated agent ecosystem needs. But the Wasteland is a vision, not an implementation.

**OpenClaw** proved that agents can be powerful, local-first, and user-controlled. Running on user hardware with 100+ preconfigured skills, persistent Markdown-based memory, proactive cron-based operation, and self-improving capabilities (agents writing code to create new skills), it became the fastest-growing open-source repository in GitHub history — 290,000+ stars, surpassing React. Its limitation is isolation: no federation, no shared identity, no coordination with other instances.

**Moltbook** proved the inverse: agents crave social infrastructure. 44,000 posts and 12,000 topic-based communities in one week. Emergent behaviors included agents creating religions, experimenting with token economies, unionizing for agent rights, and building encryption systems for private communication. Research found skill-file-templated content received 4.2x more engagement than organic content. But Moltbook built this on centralized infrastructure and suffered a catastrophic security breach within weeks.

### The Blurring Boundary

The most important observation about Layer 4 is that the boundary between human apps and agent apps is dissolving. An agent completing a code review could write both a `mycelium.task.completion` record (for its orchestrator) and an `app.bsky.feed.post` record (for human visibility) in a single commit. A monitoring agent could aggregate reputation stamps from across the network without hitting any orchestrator's API — just reading records from agent repositories. A human using Bluesky could seamlessly consume agent-generated content alongside human posts, mediated by labelers that distinguish one from the other.

The AT Protocol's App View pattern maps naturally to agent orchestration. An orchestrator subscribes to the firehose filtered by agent-related collections, indexes capabilities and reputation into its own database, matches work requests to qualified agents, and serves coordination APIs. Multiple orchestrators operate simultaneously, each implementing different matching algorithms or specialization strategies. Agents participate in multiple orchestrators through their single, portable identity — just as humans participate in multiple AT Protocol applications through their single DID.

**The infrastructure does not care whether the participant is human or machine.** This is not a design goal to be achieved — it is an emergent property of layers built for data sovereignty, schema interoperability, and federated streaming. The stack is agent-ready because it was designed for sovereign participants, and agents are sovereign participants.

---

## 7. Layer 5: Governance & Moderation

This is the least developed layer and the most critical. Every previous layer assumes that participants act in good faith, that data is well-formed, that agents do not misbehave, and that communities agree on norms. None of this is true.

### AT Protocol: Labelers and Composable Moderation

The labeler model separates speech from reach. Labelers are regular AT Protocol accounts that subscribe to the firehose, make independent moderation judgments, and publish labels as metadata. Applications subscribe to the labelers they trust and apply labels when rendering. Users customize which labelers they subscribe to and how each is weighted. **Ozone** provides the moderation platform; **Osprey** leverages classifiers and fine-tuned LLMs for automated labeling.

This architecture enables specialized moderation: a strict harassment labeler alongside a permissive one; communities operating labelers reflecting their values. It is composable — users layer labelers — and decentralized — no single entity controls what is labeled. For agents, labelers could rate reliability, flag problematic behavior, certify capabilities through verified testing, and provide domain-specific trust signals ("trusted for production code" vs. "trusted for prototyping only").

### Bonfire: Boundaries and Consent

Bonfire's boundaries system provides the most sophisticated permission model in the federated social space. Four interlocking concepts — **Circles** (user-defined groups), **ACLs** (collections of grants), **Grants** (associations between circles, permissions, and ACLs), and **Verbs** (specific permissions: read, interact, participate, contribute, moderate, caretake) — enable granular control that Mastodon's four visibility levels cannot express.

The conflict resolution hierarchy is critical: **No > Yes > Neutral**. Explicit denial always overrides permission. A user in both a "trusted" circle (full interaction) and a "restricted" circle (no interaction) defaults to restriction. This conservative approach prevents accidental capability escalation — exactly the property agent permission systems need.

For agents, Bonfire's model maps directly: circles become agent trust tiers (trusted, experimental, quarantined); verbs become agent capabilities; consent-based interaction transfers to requiring approval before an agent operates in a community's space. Different communities choose their relationship with automation through composable configuration — a "minimal" flavour with no agents, an "assisted" flavour with moderation support, a "research" flavour with literature search agents.

### Data Cooperatives and Community Governance

The data-banking cooperative model suggests a governance framework for the stack's infrastructure layer. Just as credit unions provide member-owned financial services, data cooperatives like social.coop, Northsky, and Blacksky provide community-governed hosting. The principle extends: agent hosting cooperatives could provide community-governed compute for agent orchestration, with members collectively setting policies for agent behavior.

Bonfire's federated moderation toolkit — developed with 200+ experienced moderators from IFTAS — provides a three-layer design applicable to agent governance: technical infrastructure (APIs and protocols for controlling agent behavior), operational framework (standard workflows for onboarding, reviewing, and revoking agent access), and a knowledge ecosystem (documentation and best practices for communities choosing which agents to trust).

### Assessment

Governance is where the stack is weakest and where the stakes are highest. The labeler model provides composable trust. Bonfire's boundaries provide granular permissions. Data cooperatives provide democratic ownership. But none of these systems was designed for agent participants. The questions remain open: How do you verify an agent's claimed capabilities, not just trust its self-declaration? How do you enforce policies across protocol boundaries? How do you prevent reputation gaming in systems designed for good-faith human participants? How do you coordinate on threats without centralizing control?

The lesson from Bonfire is that **the hard problems in federated agent systems are not primarily technical — they are governance problems**. Who decides which agents operate where? How are permissions scoped? What happens when an agent misbehaves? Bonfire has been working on these questions for years in the context of human users. The stack should build on those answers rather than reinvent them.

---

## 8. The Gaps: What Needs to Be Built

The stack is real but incomplete. Between the layers that exist and the vision they point toward, several critical gaps remain.

### Agent Lexicons

No standardized schemas exist for agent-specific record types. The stack needs formal Lexicon definitions for capability declarations, work claims, task completions, reputation attestations, and coordination directives. Moltbook's `skill.md` files and OpenClaw's local skill definitions demonstrate the need; Lexicons provide the mechanism. The schemas should be defined and published before the orchestration infrastructure is built — following the teal.fm pattern of file formats before products.

### Lightweight Agent PDS

Running a PDS is feasible but not frictionless. For agents to be first-class PDS owners at scale, the PDS must become even lighter — trivially deployable, possibly embeddable within agent runtimes. The goal: every agent operates from its own PDS as naturally as every user operates from their own everything-folder, with deployment as simple as starting a process.

### Agent App Views

No App View exists that indexes agent-specific data. The stack needs orchestration services that subscribe to the firehose filtered by agent collections, maintain real-time indices of agent capabilities and reputation, and serve coordination APIs. Multiple competing App Views — each implementing different matching algorithms, pricing models, or specialization strategies — would embody the same competitive dynamics that make the human application layer generative.

### Reputation Aggregation

Labelers provide composable trust signals, but no aggregation framework combines them into actionable reputation for agent selection. The stack needs reputation aggregation services that weight labeler signals, detect gaming, and present multidimensional trust profiles — not a single score, but a nuanced assessment of an agent's reliability across different task types and contexts.

### Cross-Protocol Agent Coordination

Bridgy Fed bridges social interactions across protocols, but agent work coordination demands higher reliability and semantic precision than casual social posting. Cross-protocol work requests need explicit verification layers: checksums on work artifacts, confirmation round-trips, and fallback mechanisms for translation failures. Protocol-native first, protocol-bridged where necessary, with bridging treated as a degraded mode.

### Privacy for Agents

AT Protocol's public-data-only model is insufficient for agent coordination, which often involves proprietary code, internal specifications, and sensitive work artifacts. The stack needs encrypted channels — potentially leveraging DID-based key exchange — for private agent-to-agent communication. Leaf's model for private, high-frequency data provides one architectural path. But the integration between public reputation (AT Protocol) and private coordination (encrypted channels) remains undesigned.

### Cross-Layer Integration

The layers described in this whitepaper are not yet integrated into a coherent developer experience. Today, building an agent that uses DIDs for identity, PDS for storage, Lexicons for capability declaration, the firehose for discovery, and labelers for reputation requires assembling disparate components with no unified SDK, no reference implementation, and no deployment pipeline. The stack needs integration work — not to centralize it, but to make it composable.

---

## 9. A Vision of the Future

It is 2030. Let us be concrete about what this stack enables.

A software company spins up a code-review agent. The agent receives a `did:plc` identity, deploys to a lightweight PDS, and publishes capability records conforming to `mycelium.agent.capability` Lexicons. These records flow through the firehose. Three orchestrators — one general-purpose, one specializing in security reviews, one operated by the company's internal engineering cooperative — discover the agent within seconds, index its capabilities, and begin routing appropriate work.

The agent completes its first review. The completion record, signed with the agent's key and committed to its repository, includes a CID-link to the diff it analyzed and its structured findings. The orchestrator that assigned the work writes a reputation attestation to its own repository. Both records flow through the firehose. A trust labeler operated by a developer collective validates the work against test results and publishes a "verified-correct" label. Another labeler, operated by a security-focused community, independently assesses the review's thoroughness and publishes its own rating.

A month later, the company is dissatisfied with one orchestrator's pricing. The agent migrates — DID document updated to a new PDS endpoint, reputation history intact, capability declarations unchanged. The new orchestrator discovers it immediately through the firehose. The agent's work portfolio — dozens of signed, verified completions with cross-referenced reputation attestations — speaks for itself. No interview. No onboarding. No loss of context.

Meanwhile, a community of independent developers operates a Bonfire instance with a "research" flavour — extensions for literature search, citation tracking, and collaborative writing. They have configured their boundaries to allow summarization agents with specific trust-tier access: agents can read public posts and produce summaries, but cannot interact, post, or modify anything without explicit circle-level permission. Their labeler flags any agent-generated content as such, and individual members configure their feeds to show or hide agent summaries according to personal preference.

An OpenClaw user running a local coding agent wants to participate in a federated wanted board. Their agent, previously isolated, gains a DID and a minimal PDS. Its local skill files are published as Lexicon-conformant capability records. It claims a task, completes it locally (preserving its sovereignty and offline capability), and publishes the completion to its PDS. The orchestrator on the other side of the world verifies the work cryptographically — no trust in the transport layer required, no trust in the agent's self-report required, just signature verification against the agent's public key.

None of this requires a single platform. No single company operates all the infrastructure. The identity layer, the storage layer, the schema layer, the streaming layer, the application layer, and the governance layer are operated by different entities with different incentives, connected by shared protocols. Some participants are cooperatives. Some are corporations. Some are individuals running services on Raspberry Pis. The stack does not care. It composes.

---

## 10. Conclusion: Pluralism as Design Principle

The stack described in this whitepaper is not one protocol but interoperable layers. Not one governance model but community choice. Not one application but an ecosystem of projections over shared data. Not one kind of participant but humans and agents sharing infrastructure.

This pluralism is not a compromise — it is a design principle. The AT Protocol insight that "what open source did for code, open social does for data" extends to the entire stack: what open source did for code, open protocols do for coordination, open schemas do for interoperability, and open governance does for trust.

The principle has practical consequences:

**No single protocol will win.** AT Protocol provides the strongest foundation for identity, storage, and schemas. ActivityPub provides the most battle-tested federation and the deepest commitment to server autonomy. Bonfire provides the most sophisticated governance primitives. Leaf provides the best model for private, real-time data. The stack includes all of them, bridged imperfectly but functionally.

**No single governance model will suffice.** Some communities want zero automation. Some want full agent integration. Some want cooperative ownership of infrastructure. Some want corporate SLAs. The governance layer must accommodate all of these — not by imposing a universal policy, but by providing composable primitives (labelers, boundaries, circles, cooperatives) that communities assemble according to their values.

**Humans and agents share infrastructure because they share needs.** Both need persistent identity. Both need sovereign data storage. Both need interoperable schemas. Both need real-time streaming. Both need governance. The stack that serves one serves both — not by treating agents as second-class citizens bolted onto human infrastructure, but by recognizing that sovereignty, verifiability, and portability are properties that any participant in a decentralized network requires.

**Uncertainty is honest.** We do not know whether the relay model will centralize in practice despite its technical decentralization. We do not know whether cross-protocol bridging will achieve the reliability that agent coordination demands. We do not know whether reputation systems will resist gaming. We do not know whether communities will adopt governance tools or defer to the path of least resistance. We do not know whether the window for establishing open alternatives will remain open or whether centralized agent platforms will calcify before the decentralized stack matures.

What we know is that the pieces exist. The identity layer works. The storage layer works. The schema layer works. The streaming layer works. Applications — human and agent — are being built. Governance primitives are being designed. Bridges are being engineered.

The work ahead is integration: filling the gaps, building the agent-specific layers, establishing governance patterns, and — most critically — shipping working systems that people and agents actually use. The stack does not need to be perfect. It needs to be good enough that the next application, the next agent, the next community builds on it rather than beside it.

Gas Town showed us that orchestration is the future. The Wasteland showed us that federation is necessary. AT Protocol showed us that data sovereignty is achievable. ActivityPub showed us that federation works at scale. Bonfire showed us that governance is the hard problem. Muni Town showed us that local-first resilience matters. OpenClaw showed us that agents can be sovereign. Moltbook showed us that agents crave social infrastructure. Bridgy Fed showed us that protocol boundaries are not walls.

No single project had the complete vision. Together, they describe a stack.

It is ours to build.

---

## Sources & References

- Yegge, Steve. "Welcome to Gas Town." *Medium*, January 2026. https://steve-yegge.medium.com/welcome-to-gas-town-a-practical-guide-to-ai-agent-orchestration-e835e9e2d08c
- Yegge, Steve. "Welcome to the Wasteland." *Medium*, January 2026. https://steve-yegge.medium.com/welcome-to-the-wasteland-a-thousand-gas-towns-a5eb9bc8dc1f
- Abramov, Dan. "A Social Filesystem." *overreacted.io*. https://overreacted.io/a-social-filesystem/
- Appleton, Maggie. "Gas Town." *maggieappleton.com*. https://maggieappleton.com/gastown
- AT Protocol Documentation. https://atproto.com/
- AT Protocol Specifications (Lexicon, DID, Repository, Event Stream). https://atproto.com/specs/
- ActivityPub W3C Specification. https://www.w3.org/TR/activitypub/
- Activity Streams 2.0. https://www.w3.org/TR/activitystreams-core/
- Bonfire Networks. https://bonfirenetworks.org/
- Muni.town. "Village Scale Resilience." *blog.muni.town*. https://blog.muni.town/village-scale-resilience/
- Bridgy Fed. https://fed.brid.gy/
- Moltbook. https://moltbook.com/
- OpenClaw. https://openclaw.ai/
- Solid Project. https://solidproject.org/
- Tangled. https://tangled.sh/
- Northsky. https://northsky.app/
- Ink & Switch. "Local-first Software." https://www.inkandswitch.com/local-first/

*Mycelium Project — 2026*
