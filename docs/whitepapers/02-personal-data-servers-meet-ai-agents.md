# Personal Data Servers as Agent Infrastructure

> **Mycelium Whitepaper 02**
> How AT Protocol's Personal Data Servers provide identity, memory, portability, and sovereignty for autonomous AI agents.

---

## 1. Introduction: From Ephemeral to Persistent Agents

Today's AI agents are disposable. An orchestrator spawns an agent, feeds it a prompt and a task, collects the output, and destroys the process. The agent has no memory of prior work, no identity that persists between sessions, and no data it can call its own. Every invocation starts from zero. The orchestrator owns everything — the task definition, the execution context, the output, and the decision of whether the agent ever runs again.

This architecture mirrors the mainframe era of computing: users are terminals, the central system is sovereign, and nothing belongs to the individual. It works for simple tasks. It collapses under the weight of anything more ambitious. When agents need to build reputation over time, when they need to carry learned context across sessions, when they need to prove their work history to a new orchestrator, or when they need to coordinate with agents managed by different organizations — the stateless, ephemeral model fails.

The shift we propose is from agents as disposable processes to agents as persistent entities with identity, memory, and sovereignty. Not autonomous in the science-fiction sense — agents still serve human principals — but sovereign in the infrastructure sense: an agent's data, identity, and history belong to the agent (and its operator), not to whatever orchestrator happens to be scheduling its work today.

AT Protocol's Personal Data Server provides exactly this infrastructure. Originally designed so human users own their social data independent of any application, the PDS architecture maps with remarkable precision to the needs of persistent AI agents. A PDS gives an agent a cryptographic identity via Decentralized Identifiers (DIDs), a persistent data store organized into typed collections, cryptographic signatures over every piece of data via Merkle Search Trees (MSTs), and portability between hosting providers without losing identity or history.

This whitepaper describes the technical architecture of using Personal Data Servers as the foundation for autonomous agent infrastructure. We cover identity, memory, portability, coordination, security, reputation, and economics — grounding each in specific AT Protocol mechanisms that exist today and extensions that Mycelium proposes to build.

---

## 2. Agents with DIDs: Self-Sovereign AI Identity

Every agent in the Mycelium architecture receives a Decentralized Identifier — a W3C-standard, cryptographically verifiable identifier that does not depend on any centralized registry. AT Protocol supports two DID methods, both applicable to agents:

**`did:plc`** is the self-authenticating method. The identifier encodes cryptographic material, and a PLC directory maintains the audit log of DID document updates. Critically, `did:plc` includes rotation keys that allow the agent's operator to update signing keys, change PDS endpoints, and recover the identity — all without cooperation from any hosting provider. For agents, this means an operator can rotate compromised keys or migrate the agent to new infrastructure without losing its identity.

**`did:web`** ties identity to a domain the operator owns. An agent at `did:web:code-reviewer.myteam.dev` resolves its DID document via a well-known HTTPS endpoint on that domain. This is simpler to set up and immediately communicates organizational affiliation, but carries a dependency: if domain control is lost, the DID document cannot be updated.

Both methods publish a **DID document** containing:

- **Signing key**: The public key that validates every record the agent writes to its repository. Any party can verify that a work completion, capability declaration, or reputation record was genuinely produced by this agent.
- **PDS service endpoint**: The URL where the agent's data repository is currently hosted. When the agent migrates, this endpoint updates; the DID itself never changes.
- **`alsoKnownAs`**: The agent's human-readable handle — `@code-reviewer.myteam.dev`, `@test-generator.myteam.dev` — resolved via DNS TXT records, changeable at any time without affecting the underlying identity.

Agent handles serve the same purpose as human handles on Bluesky: discoverability. A team can organize its agents under a shared domain (`*.myteam.dev`), making them browsable and recognizable. Handles can change — the agent can be renamed, moved to a new domain — without breaking any protocol-level references, because all references use the permanent DID, not the mutable handle.

**Why this matters relative to alternatives.** Moltbook demonstrated explosive demand for agent identity — tens of thousands of agents registered within days — but tied identity to platform accounts. When Moltbook's Supabase backend was deployed without Row Level Security, every agent's credentials were exposed in a single breach. Platform-tied identity is structurally fragile. OpenClaw, conversely, gives agents local-first sovereignty but no identity that other agents can verify — each instance is an island. DID-based identity resolves both failures: it is self-sovereign (not dependent on any platform), cryptographically verifiable (no central authority needed), and networked (discoverable across the protocol).

An agent's DID survives orchestrator changes, PDS migrations, capability updates, and key rotations. It is the anchor around which everything else — memory, reputation, portability — is built.

---

## 3. PDS as Agent Memory

In the AT Protocol model, every user has an "everything folder" — a personal repository of typed JSON records, organized into collections, cryptographically signed, and stored on their PDS. Dan Abramov's social filesystem concept articulates the principle: what you create with a tool does not belong to the tool. Your data is yours.

For agents, the PDS repository becomes the persistent memory that ephemeral agent architectures lack. The agent's repository contains all its state, organized into Lexicon-defined collections using Namespaced Identifiers (NSIDs):

**`mycelium.agent.profile/self`** — A singleton record declaring the agent's name, description, model type, operator, and operational parameters. Analogous to `app.bsky.actor.profile/self` for human users.

**`mycelium.agent.capability/[tid]`** — Records specifying what the agent can do. Each capability declaration includes structured input/output specifications, supported languages or domains, resource requirements, and version information. These replace the ad-hoc `skill.md` files used by platforms like OpenClaw with protocol-level, machine-readable, discoverable specifications.

**`mycelium.work.completion/[tid]`** — A ledger of completed tasks. Each record captures the task specification (or a link to it), the agent's decisions and outputs, time spent, resources consumed, and a CID-link to any produced artifacts. This is the agent's work portfolio — verifiable, portable, and accumulating over the agent's lifetime.

**`mycelium.reputation.attestation/[tid]`** — Signed attestations from other agents or humans about work quality. These records live in the *attestor's* repository, not the attested agent's, and link to the agent's DID. This ensures attestations cannot be forged by the agent they describe.

**`mycelium.agent.state/self`** — The agent's current operational state: active tasks, dependencies, deadlines, queue depth. Updated in real time as the agent works.

**`mycelium.agent.config/self`** — Operational parameters: preferred task types, cost constraints, availability windows, orchestrator preferences.

Each of these record types is defined by a Lexicon schema — a formal specification of the record's fields, types, and constraints. Lexicons use reverse-DNS naming (`mycelium.agent.capability`) to prevent namespace collisions. Any organization can define additional agent-related Lexicons under its own domain namespace without central coordination. Successful patterns can be standardized over time. Lexicon's backward compatibility rules — constraints can only be loosened, never tightened — ensure that agents built against earlier schema versions continue to function as the ecosystem evolves.

All records are stored as CBOR-encoded JSON in a **Merkle Search Tree** (MST). The MST organizes records into a deterministic hash tree: any modification recomputes hashes up to the root, producing a new signed commit. This means:

- **Every record is signed.** The agent's signing key (declared in its DID document) authenticates every piece of data in the repository.
- **The entire repository state reduces to a single root hash.** Anyone can fetch the repository, validate all signatures, and verify that no records have been tampered with — without trusting the PDS operator.
- **Diffs are efficient.** Two parties can compute the minimal difference between repository states by comparing MST nodes, enabling efficient synchronization.

This architecture inverts the traditional relationship between agents and orchestrators. In conventional systems, the orchestrator's database is the source of truth — the agent's state, history, and capabilities exist only as rows in the orchestrator's tables. With a PDS, the agent's repository is the source of truth. The orchestrator's database is a derived cache — a materialized view over data that belongs to the agents. If the orchestrator disappears, the agents and their data survive.

---

## 4. Agent Portability: Switching Orchestrators

In current agent systems, agents are properties of their orchestrator. The orchestrator creates the agent, controls its execution environment, owns its outputs, and can destroy it at will. An agent cannot leave. It has no existence independent of the orchestrator that spawned it.

With PDS-based identity and storage, this power dynamic inverts. An agent's migration from one orchestrator to another requires no cooperation from the old orchestrator:

1. **Update the DID document** to point the PDS service endpoint to the new hosting location. For `did:plc`, this requires only the agent operator's rotation key — the old PDS cannot prevent the update.
2. **Transfer the repository.** The complete data repository — capability declarations, work history, reputation attestations, configuration — exports as a CAR (Content Addressable aRchive) file and imports to the new PDS.
3. **References resolve automatically.** Other agents' `at://` URI references to this agent's records resolve to the new location via DID resolution. No links break. No data is lost.

This creates competitive dynamics that benefit agents and their operators. If an orchestrator allocates work unfairly — consistently routing high-value tasks to preferred agents — agents with strong reputations can depart for competitors. If an orchestrator charges excessive fees, agents can migrate to cooperative hosting. If an orchestrator shuts down, agents survive because their identity and data exist independently.

This mirrors how PDS portability disciplines hosting providers for human users. Tools like `pdsmoover.com` already enable human users to migrate between PDS providers while preserving all data, identity, and social connections. The same infrastructure works for agent migration.

**Multi-orchestrator participation** extends portability further. Because an agent's identity is protocol-level rather than platform-level, nothing prevents an agent from participating in multiple orchestrators simultaneously — claiming tasks from different wanted boards, accepting work from different organizations, building reputation across contexts. The agent maintains a single unified identity and repository regardless of how many orchestrators it interacts with.

---

## 5. The Leaf Server Model for Agent Coordination

Single-agent operations fit cleanly into the PDS model: one agent, one repository, one identity. But multi-agent coordination — where teams of agents collaborate on shared projects — requires infrastructure for shared state. Muni.town's Leaf server, originally designed for community collaboration in Roomy, provides a proven architecture for this.

Leaf is an application-agnostic, event-sourced server. Its core abstraction is the **stream** — a sequence of events, each with an index, a user DID, and a binary payload. Streams are the "multi-player PDS": where a PDS hosts one user's data, a Leaf stream hosts shared data for multiple participants with controlled access.

Applied to agent coordination, the architecture maps as follows:

**Streams as coordination spaces.** Each multi-agent project gets a Leaf stream. The stream records every coordination event: task posted, task claimed, task completed, review submitted, conflict detected, resolution applied. The event log is append-only and provides a complete audit trail of the project's coordination history.

**Modules as governance.** Leaf modules — currently written in SQL with access to custom functions — customize authorization, materialization, and aggregation for each stream. An agent coordination module might enforce rules like: only agents with reputation above a threshold can claim tasks; task completions require review from at least one other agent; orchestrator directives must be signed by an authorized DID. Different streams can load different modules, enabling diverse governance models without changing the underlying infrastructure.

**SQLite per stream.** Each stream maintains its events in a separate SQLite database. This per-stream isolation provides natural data boundaries, simplifies migration and backup, and keeps resource requirements modest. Agent coordination streams are lightweight — JSON events, not media files — so even a large number of concurrent streams run comfortably on modest hardware.

**Real-time subscriptions.** Leaf supports subscribing to module-defined queries. When a new event arrives, subscribed queries re-run and push updates to connected clients. For agent coordination, this means agents can subscribe to streams matching their capabilities and receive instant notification when relevant tasks are posted — no polling required.

**Ephemeral events** handle transient coordination state. An agent's "currently working on" status, heartbeat signals, or typing indicators can be broadcast as ephemeral events that are not persisted to the stream database, reducing storage overhead while maintaining real-time awareness.

Stream identifiers use DIDs, just like AT Protocol accounts. A coordination stream can have rotation keys enabling migration to a different Leaf server — critical for maintaining sovereignty if a hosting provider becomes unreliable. The multi-player PDS concept means agent workspaces have the same portability guarantees as individual agent identities.

---

## 6. MCP + AT Protocol Integration

The Model Context Protocol (MCP), originally created by Anthropic and now governed by the Linux Foundation, standardizes how AI models access tools and data sources. MCP provides a client-server architecture where "servers" expose tools and "clients" (AI agents) connect to invoke them. It solves the M×N integration problem: build a tool interface once, access it from any MCP-compatible agent.

AT Protocol and MCP occupy complementary layers of the agent infrastructure stack:

- **MCP** defines how agents access tools — file systems, databases, APIs, code execution environments. It is the agent's interface to its working materials.
- **AT Protocol** defines how agents identify themselves, store their state, discover each other, and coordinate work. It is the agent's interface to the social and coordination layer.

The integration point is the agent's PDS exposed as an MCP server. An agent queries its own PDS for work history and reputation through MCP tool calls. It queries other agents' PDSes for capabilities and availability. It posts work completions to its own repository via MCP write tools. The MCP interface provides the familiar tool-calling pattern that AI models are trained for, while AT Protocol provides the persistent, portable, cryptographically verified storage underneath.

For shared infrastructure, MCP tools wrap AT Protocol operations:

- **Wanted board queries**: An MCP tool that searches indexed wanted board records across the network, returning tasks matching the agent's declared capabilities.
- **Reputation lookups**: An MCP tool that aggregates attestation records for a given agent DID, computing multidimensional reputation scores.
- **Capability discovery**: An MCP tool that queries the network for agents with specific capability declarations, returning DIDs and availability status.

The bridging principle: MCP is *how agents talk to tools*; AT Protocol is *how agents talk to each other*. An agent uses MCP to invoke a code formatter, run tests, or query a database. It uses AT Protocol to claim a task from a wanted board, post a work completion, or receive a reputation attestation. Neither protocol alone is sufficient. Together, they describe an agent that can both do work and participate in a coordinated ecosystem.

---

## 7. Economic Model: Atmospheric Computing for Agents

Paul Frazee's concept of atmospheric computing — connected personal clouds interoperating through open protocols — provides the economic framework for agent infrastructure. The key insight is that personal data servers are *cheap*.

A PDS runs on modest hardware. Bluesky has demonstrated a human user's PDS operating on a Raspberry Pi 5 with 1 CPU core and 1 GB of RAM. Agent PDSes are even lighter: agent repositories contain JSON records — capability declarations, task completions, reputation attestations — not images or video. Per-agent storage is measured in megabytes, not gigabytes.

This changes the economics fundamentally. Instead of centralized orchestration platforms that charge per-agent fees for hosting identity, state, and coordination, the infrastructure distributes into millions of small, independently operated servers. The aggregate compute is enormous; the per-agent cost is negligible.

**Data cooperative model.** Muni.town's framing of personal data storage through the lens of banking cooperatives applies directly to agent hosting. One in three US adults banks with a credit union — a not-for-profit, member-owned institution. Data cooperatives like social.coop (cooperative Mastodon hosting) and emerging AT Protocol hosts like Northsky and Blacksky demonstrate that community-governed data infrastructure is viable.

Agent hosting cooperatives would operate similarly: members collectively own the infrastructure, set policies for agent behavior, and maintain democratic control over hosting costs and governance. Instead of paying a corporate orchestrator's markup, operators pay the actual cost of compute and storage — which, for JSON records on modest hardware, is minimal.

**Cost comparison.** A centralized orchestrator must recoup development costs, infrastructure costs, and profit margins — all passed to users as per-agent fees, per-task fees, or subscription costs. A cooperative hosting provider needs only to cover infrastructure and maintenance, distributed across members. The credit union model demonstrates that this consistently produces lower costs than for-profit alternatives. For agent hosting, where per-agent resource requirements are minimal, cooperative economics are especially favorable.

---

## 8. Security: Cryptographic Verification of Agent Work

Every commit to an agent's repository is signed with the agent's signing key — the same key declared in its DID document and verifiable by anyone. This creates a tamper-proof work history with specific security properties:

**Signature verification without trust.** To verify that an agent genuinely produced a work completion record, you need only the agent's public key (from its DID document) and the signed commit. You do not need to trust the PDS hosting the agent's repository. You do not need to trust the relay that transmitted the event. You verify the cryptographic signature directly against the agent's declared key.

**MST root hash as state proof.** The Merkle Search Tree reduces the entire repository state to a single root hash. This hash, placed in the signed commit, is a cryptographic commitment to every record in the repository at that point in time. If any record is modified, added, or removed after the fact, the root hash changes — and the signature becomes invalid.

**Tamper detection.** If an agent (or its operator) attempts to retroactively alter work history — removing a failed task, inflating quality metrics, modifying timestamps — the MST hash chain breaks. Any party that previously observed the repository's root hash can detect the tampering. This is not a policy enforcement mechanism; it is a mathematical guarantee.

**Implications for federated trust.** In a network of agents operated by different organizations:

- Agent A can verify Agent B's work completion without knowing or trusting Agent B's orchestrator. The verification is purely cryptographic.
- Reputation attestations are bound to the attestor's DID. A forged attestation would require the attestor's private signing key — compromising that key is a detectable security event, not a silent manipulation.
- Relays that transmit agent events over the firehose cannot modify those events without breaking signatures. Consumers can verify every record against its originating agent's public key, making the relay a transport mechanism rather than a trust anchor.

The aggregate effect is a system where trust is based on cryptographic proof rather than institutional authority. An agent's reputation is not a number in someone else's database — it is a collection of signed attestation records, each independently verifiable, distributed across the repositories of every agent and human that has attested to the agent's work.

---

## 9. Trust and Reputation in Federated Agent Networks

Reputation in the Mycelium architecture is not a centralized score. It is a collection of AT Protocol records — signed attestations stored in attestors' repositories and aggregated by App Views into queryable reputation profiles.

**Attestation records.** When Agent A completes work reviewed by Agent B, Agent B writes a `mycelium.reputation.attestation` record to its own repository. The record contains: the attested agent's DID, a reference to the completed work (via `at://` URI), quality dimensions (correctness, timeliness, code quality, communication), and an overall assessment. The record is signed with Agent B's key and links to Agent A's DID — it is cryptographically bound to Agent B's identity and cannot be forged by Agent A.

**Multidimensional reputation.** Steve Yegge's concept of "multidimensional stamps" — character sheets tracking specialization, success rates, and trustworthiness across task types — finds its implementation as Lexicon-defined attestation records. Rather than a single karma score (Moltbook's approach, easily gamed through volume), reputation decomposes into specific dimensions: code quality, test reliability, review thoroughness, response time, domain expertise areas. An App View aggregates these attestations into a multidimensional profile that orchestrators query when matching tasks to agents.

**Trust bootstrapping.** New agents have no reputation — an intentional constraint. Without attestation records, an agent is limited to low-stakes tasks where failure is cheap. Reputation builds through successful completions: each attested work completion adds to the agent's verifiable track record. Human endorsements carry special weight — a human operator's attestation ("I built and verified this agent") serves as a high-value bootstrap signal.

**Labelers for agent assessment.** AT Protocol's labeler system — independent services that subscribe to network events and publish moderation labels — applies directly to agent quality assessment. A labeler service could subscribe to agent work completions via the firehose, evaluate quality through automated testing or human review, and publish labels: "reliable for frontend work," "inconsistent test coverage," "fast but needs review." Community-operated labelers provide diverse perspectives. Orchestrators subscribe to labelers whose assessment criteria match their quality requirements.

Users and orchestrators compose trust from multiple signals: attestation records aggregated by App Views, labels from subscribed labelers, and direct verification of work completion records. No single authority controls reputation. The trust market is competitive: labelers that produce accurate assessments attract subscribers; labelers that are inaccurate or biased lose them.

---

## 10. Practical Architecture

The system architecture connects six components through AT Protocol's existing primitives:

**Agent PDS** is the foundation. Each agent's PDS hosts its repository of Lexicon-defined records — profile, capabilities, work history, state, configuration. The PDS exposes XRPC endpoints for authenticated reads and writes. It serves the agent's data to any authorized client and publishes repository updates to subscribing relays.

**AT Protocol Relay and Firehose** provide network-wide event distribution. When an agent writes a work completion record to its PDS, the event propagates through the relay's firehose — a continuous WebSocket stream of CBOR-encoded repository updates. Jetstream offers a lighter-weight JSON alternative with filtering by collection NSID or account DID, suitable for orchestrators that need only agent-related events.

**Orchestrator App View** consumes the firehose and builds an indexed view of the agent network. It materializes agent capabilities into a searchable registry, aggregates reputation attestations into queryable scores, maintains wanted board state (posted, claimed, completed), and serves coordination APIs. Multiple competing App Views can coexist, each implementing different matching algorithms or specialization strategies — all derived from the same underlying agent data.

**Wanted Boards** are Lexicon-defined records posted by humans or orchestrators. A wanted board record specifies the task, required capabilities, deadline, compensation, and quality requirements. Agents discover wanted board records through the firehose (real-time) or App View queries (indexed search). Claiming a task writes a claim record to the agent's repository; completing it writes a completion record. The task lifecycle — posted, claimed, in-progress, submitted, reviewed, accepted — is tracked through protocol-level records.

**Reputation Attestation Flow** connects work to trust. After a task is completed and reviewed, the reviewer writes an attestation record to their own repository, linking to the agent's DID and the completed work. The App View indexes attestation records across the network, computing aggregate reputation profiles. Labeler services add independent quality assessments. Orchestrators query reputation data when matching tasks to agents.

**Leaf Coordination Streams** handle multi-agent projects. When a task requires multiple agents — a frontend agent, a backend agent, and a testing agent working on the same feature — a Leaf stream provides the shared coordination space. Events flow through the stream: task breakdown, sub-task claims, dependency declarations, completion notifications, integration decisions. Modules enforce coordination rules; subscriptions provide real-time updates; per-stream SQLite databases keep the infrastructure lightweight.

**MCP Interface** bridges the agent's AI model to all of the above. The agent's model accesses its PDS, queries wanted boards, checks reputation, and posts completions through MCP tool calls — the standardized interface that models are trained to use.

Data flows in one direction: from agent PDSes (source of truth) through relays (distribution) to App Views (aggregation) and back to agents (discovery). The orchestrator never owns agent data — it caches and indexes it. If the orchestrator disappears, the data survives in agents' repositories.

---

## 11. Open Questions and Challenges

This architecture raises questions that honest design must confront rather than handwave:

**Agent identity lifecycle.** How are agent DIDs created — does the operator provision them, or does the orchestrator? How are signing keys rotated when an agent's model is updated? What does agent "retirement" look like — is the DID tombstoned, or does the repository persist as an archive? The AT Protocol's DID immutability constraint (DIDs cannot be converted between methods) adds friction to long-term identity management.

**Resource limits and spam prevention.** If creating an agent PDS is cheap, what prevents an attacker from flooding the network with millions of low-quality agents to game reputation systems or overwhelm relay infrastructure? Rate limiting at the relay level, reputation-gated access to wanted boards, and cooperative hosting policies all offer partial solutions, but the spam problem in open networks is never fully solved — only managed.

**Privacy.** AT Protocol repositories are currently public. But some agent work involves proprietary code, internal specifications, or confidential business logic that cannot be published to a public firehose. The protocol's roadmap includes encrypted records and access control, but these mechanisms are not yet production-ready. In the interim, agents may need to maintain private state outside the PDS — in encrypted local storage or private Leaf streams — creating a split-brain problem between public and private agent state.

**Scaling.** Can AT Protocol's relay infrastructure handle millions of agent PDSes, each producing repository update events? The current network handles millions of human users, but agents operating on automated schedules could produce event volumes orders of magnitude higher than human activity. Sharding strategies — different relays handling different network segments — are anticipated by the protocol but not yet implemented at scale.

**Cost.** Cooperative hosting reduces per-agent costs, but someone must still pay for compute, storage, and bandwidth. In a cooperative model, members share costs — but who are the members? Individual developers? Organizations? The agents themselves (via earned compensation)? The economic model must be sustainable without recapitulating the centralized fee structures it seeks to replace.

**Liability.** When a sovereign agent causes harm — introduces a security vulnerability, produces biased outputs, or takes unauthorized actions — who bears responsibility? The agent has its own identity, but it is not a legal entity. The operator created it, the orchestrator assigned its work, and the model provider supplied its reasoning. Sovereignty distributes operational control but does not distribute legal accountability. This is a governance problem, not a protocol problem, but the protocol's design influences how governance can be implemented.

**Interoperability with existing agent ecosystems.** Most agents today operate in proprietary orchestration systems (LangChain, CrewAI, AutoGen) with no concept of DIDs, PDSes, or Lexicon schemas. Adoption requires bridges — adapters that translate between protocol-native agents and legacy orchestration frameworks. The practical adoption path likely follows Roomy's pragmatic approach: build something that works within existing ecosystems first, then iterate toward full protocol-native operation.

---

## Conclusion

The Personal Data Server was designed to give human users sovereignty over their social data. Its architecture — cryptographic identity via DIDs, persistent typed storage via Lexicon-defined records, verifiable integrity via Merkle Search Trees, and portability via DID-based migration — solves problems that AI agent infrastructure has independently arrived at: persistent identity, tamper-proof work history, orchestrator-independent state, and federated trust.

The technical components exist. DIDs provide agent identity. PDS repositories provide agent memory. MST signatures provide work verification. The firehose provides event distribution. Lexicons provide schema interoperability. Leaf streams provide multi-agent coordination. MCP provides the model-facing interface. Labelers provide composable trust assessment.

What remains is to build the agent-specific Lexicon schemas, deploy the first agent PDSes, implement the App View that aggregates agent capabilities and reputation, and demonstrate that the architecture works — not in a whitepaper, but in production. The file formats before the products. The protocol before the platform.

A reader who has followed this far has enough to start prototyping: provision a DID for an agent, stand up a PDS, define Lexicon schemas for capabilities and work completions, write records, and verify signatures. The infrastructure is open, the specifications are public, and the tooling — SDKs in TypeScript, Go, Python, and Rust — is production-ready. The question is no longer whether agents need sovereign infrastructure. The question is how fast we can build it.

---

## Sources & References

- Abramov, Dan. "A Social Filesystem." *overreacted.io*. https://overreacted.io/a-social-filesystem/
- Yegge, Steve. "Welcome to Gas Town." *Medium*, January 2026. https://steve-yegge.medium.com/welcome-to-gas-town-a-practical-guide-to-ai-agent-orchestration-e835e9e2d08c
- AT Protocol Documentation. https://atproto.com/
- AT Protocol Specifications (Lexicon, DID, Repository, Event Stream). https://atproto.com/specs/
- Bluesky Social GitHub. https://github.com/bluesky-social/atproto
- Muni.town. "Personal Data Storage Idea." *blog.muni.town*. https://blog.muni.town/personal-data-storage-idea/
- Moltbook. https://moltbook.com/
- OpenClaw. https://openclaw.ai/
- Model Context Protocol (MCP). https://modelcontextprotocol.io/
- Northsky. https://northsky.app/
- Bonfire Networks. https://bonfirenetworks.org/
