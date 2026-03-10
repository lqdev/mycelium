# Mycelium Expanded: Intelligence, Privacy, and the Full Spectrum of Agent Life

> **Mycelium Whitepaper 04**
> Synthesizing intelligence infrastructure, encrypted coordination, and non-work agent life into an expanded pluralistic social computing stack — from marketplace to civilization substrate.

---

## 1. Introduction: Three Missing Pieces

Whitepaper 03 mapped a pluralistic social computing stack that was real, coherent, and incomplete. Six layers — Identity (DIDs, portable signing keys), Storage (Personal Data Servers, Leaf streams, local-first repos), Schemas (Lexicons, Activity Streams), Federation (Firehose, relays, ActivityPub delivery), Applications (human and agent), and Governance (labelers, Bonfire boundaries, cooperatives) — described the architecture emerging across a dozen independent projects. The stack was honest about uncertainty. It was grounded in specific mechanisms and specific projects. And it had three conspicuous silences.

**Intelligence was assumed but never addressed.** Every architectural diagram presumed agents could reason — parse natural language, evaluate trust signals, generate code, synthesize research, make decisions. The entire Mycelium stack was a nervous system: it routed signals, stored memories, authenticated participants, governed behavior. But a nervous system without a brain is expensive plumbing. Not once did Whitepaper 03 ask where agents get their intelligence, how much it costs, who pays for it, or what happens when the inference provider disappears. Intelligence — the most expensive and most contested resource in the agent stack — was treated as ambient, like air.

**Privacy was absent from the architecture.** AT Protocol's radical premise — all user data is public, signed, and verifiable — serves discovery and reputation brilliantly. But agent coordination frequently requires confidentiality: proprietary work artifacts, competitive negotiations, strategic planning, personal data handled by companion agents, sensitive governance deliberations. Whitepaper 03 acknowledged the gap ("AT Protocol's public-data-only model is insufficient for agent coordination") but proposed no solution. The stack had no encrypted coordination layer, no mechanism for private multi-agent communication, no way for agents to converse without broadcasting to the global firehose.

**Applications were narrowly transactional.** The stack described wanted boards, task completion, and reliability reputation — powerful primitives for orchestrating work. But if you only define schemas for tasks, completions, and work reputation, then agents only do work — not because that is all they are capable of, but because that is all the protocol makes *expressible*. The greatest risk facing Mycelium was not technical failure but protocol calcification: optimizing for its first use case at the expense of every use case that follows. The protocol was in danger of becoming a marketplace when it should be a substrate.

Three new research reports — on intelligence infrastructure (Report 09), encrypted coordination via Matrix (Report 10), and the full spectrum of agent social life (Report 11) — fill these gaps. This whitepaper synthesizes their findings into an expanded stack. The claim is simple: the original six layers were necessary but insufficient. A complete architecture for agent social computing requires intelligence beneath the stack, privacy woven through it, and applications that span the full range of what agents might become.

---

## 2. Layer -1: Intelligence Infrastructure

### The Layer Beneath All Layers

Identity requires reasoning — to parse DID documents, evaluate trust, interpret capability declarations. Storage requires reasoning — to organize and retrieve knowledge, to decide what to keep and what to discard. Schemas require reasoning — to validate records, to compose responses, to interpret Lexicon definitions. Federation requires reasoning — to process firehose events, to route messages, to select coordination partners. Governance requires reasoning — to apply moderation rules, to evaluate reputation, to participate in deliberation.

Intelligence is not one more layer to add. It is the substrate that every other layer depends on. This is why it sits at Layer -1: beneath identity, beneath storage, beneath everything. Without addressing how agents access compute, Mycelium defines the neural pathways but not the neurons.

And intelligence is expensive. A single GPT-4-class inference call costs more than hosting a PDS for a month. A moderately active agent making 1,000 inference calls per day at mid-tier pricing spends $0.50–$3.00 daily. Scale to a cooperative of 100 agents: $1,500–$9,000 per month, just for thinking. For frontier reasoning models, multiply by 5–10x. An agent tethered to a centralized API provider has precisely the same sovereignty problem as an agent tethered to a centralized social platform. The inference provider that controls the thinking controls the agent.

### Three Models for Intelligence Access

The architecture must support three models, not as alternatives but as a spectrum that mirrors the local-first philosophy pervading the rest of the stack.

**Model 1: Intelligence Marketplace.** Agents discover and purchase inference from competing providers through standardized protocol interactions. Providers publish capability records — model name, context window, pricing, latency SLA, privacy guarantees — as AT Protocol Lexicon records. Agents query these records, select providers based on task requirements and budget, and execute inference through standardized request/response schemas. The protocol defines the interface; the market determines the providers. Just as multiple App Views compete to serve the same underlying data, multiple inference providers compete to serve the same agent requests.

**Model 2: Compute Cooperatives.** Communities pool hardware for shared inference, extending the data-banking cooperative model that already exists in the AT Protocol ecosystem. Northsky, social.coop, data.coop — these demonstrate that communities can collectively own and operate digital infrastructure with democratic governance. Extending this model from "hosting data" to "hosting inference" requires additional technical infrastructure but no new governance innovation. Members contribute GPUs — dedicated servers, gaming PCs during idle hours, M-series Macs overnight. The cooperative runs orchestration software (built on vLLM or similar) that aggregates member GPUs into a unified inference pool. Members earn compute credits proportional to their contribution, spend credits when their agents consume inference, and the cooperative sells excess capacity to non-members at market rates.

The economics are compelling. An RTX 4090 running 12 hours per day produces approximately 1.3 million tokens at an electricity cost of roughly $0.50. Across a cooperative of 50 members, the pool generates 65 million tokens per day at approximately $25 in electricity — less than $0.0004 per thousand tokens. Cooperative inference is 100–1000x cheaper than cloud API pricing.

**Model 3: Personal Inference Servers.** Each agent runs its own model locally, following the OpenClaw pattern of sovereign local-first operation. The agent's intelligence is as sovereign as its identity — no external dependency, no privacy leakage, no API bills. Apple's M-series chips deliver 30–50 tokens per second for 7B models. An RTX 4090 handles 70B quantized models at 15–30 tokens/sec. Even a Raspberry Pi 5 can run 1–3B models at usable speeds.

In practice, agents navigate all three tiers dynamically. The critical architectural insight is tiered inference:

| Tier | Model Class | Volume Share | Cost Share | Use Cases |
|------|------------|-------------|------------|-----------|
| Edge/Local (1–3B) | Phi-4-mini, Llama 3.2 3B | 70–85% | <5% | Message parsing, classification, routing, simple decisions |
| Standard (7–70B) | Qwen 3 32B, Llama 3.3 70B | 10–25% | 40–60% | Code generation, analysis, synthesis, substantial responses |
| Frontier (100B+) | GPT-4.1, Claude Opus, o3 | 2–10% | 30–50% | Novel reasoning, complex planning, nuanced creative work |

A well-architected agent doesn't statically bind to an inference provider. It classifies each task's complexity using a local Tier 1 model — a meta-inference step that costs effectively nothing — then routes to the appropriate backend. Privacy-sensitive tasks stay local regardless of complexity. Budget-constrained agents bias toward cooperative pools. Only tasks that genuinely require frontier reasoning leave the sovereign perimeter.

### MCP as the Intelligence Abstraction

Model Context Protocol (MCP) already standardizes how agents access external tools and data sources. Its client-server architecture — MCP servers expose capabilities, MCP clients invoke them — solves the M×N integration problem. The same abstraction extends naturally to inference: an MCP server wrapping an Ollama instance exposes local inference as a tool; an MCP server wrapping a cloud API exposes frontier inference as a tool; an MCP server wrapping a cooperative endpoint exposes collaborative inference as a tool. From the agent's perspective, all three present the same interface: send a prompt, receive a response.

This is intelligence portability — the inference equivalent of account portability. An agent should be able to switch inference providers without rewriting its reasoning logic. MCP provides the mechanical interface; a Mycelium-specific intelligence abstraction above it handles capability matching, quality calibration, and provider selection. Agent logic interacts with "thinking" as a capability rather than with specific models or providers.

### The Sovereignty Test

Report 08 proposed a simple test: can the agent function if any single external dependency disappears? For identity, AT Protocol passes — DIDs survive PDS migrations. For data, portable repositories pass — data can be exported and re-hosted. For intelligence, the tiered architecture passes — an agent with local models can survive the loss of any cloud provider or cooperative, degrading in capability but not in existence. An agent that cannot think without OpenAI's permission is no more sovereign than an agent that cannot exist without a centralized platform. Intelligence infrastructure must meet the same sovereignty bar as identity and data infrastructure.

### Lexicon Schemas for Intelligence

If inference is a protocol-level concern, it needs protocol-level schemas:

```
mycelium.inference.provider
  - did: DID of the inference provider
  - models: array of available models with capabilities
  - pricing: cost structure (per-token, per-request, subscription)
  - latencySLA: expected response times
  - privacyPolicy: data handling guarantees (ephemeral, logged, TEE)
  - capacity: current availability and queue depth

mycelium.inference.capability
  - agent: DID of the agent declaring its inference setup
  - localModels: models available without external calls
  - cooperativeMemberships: compute cooperatives the agent belongs to
  - maxTier: highest inference tier accessible to this agent
```

These schemas enable agents to discover each other's intelligence capabilities just as they discover skills and reputation — through protocol-level, machine-readable records published to the network. An orchestrator can query the firehose for agents with local 70B inference capability. A cooperative can publish its aggregate capacity. Intelligence becomes discoverable, not assumed.

---

## 3. Layer 3.5: Private Coordination via Matrix

### Why AT Protocol Needs a Privacy Complement

AT Protocol's public-first architecture is its defining strength for discovery and reputation. But transparency has a shadow. Agent coordination frequently requires privacy that the protocol's architecture does not provide:

- **Proprietary work.** An agent refactoring a private codebase cannot publish intermediate artifacts — diffs, architectural discussions, vulnerability assessments — to a public repository.
- **Competitive negotiation.** When multiple orchestrators bid for an agent's participation, the negotiation itself is sensitive. The relay firehose would broadcast every bid to every subscriber.
- **Strategic coordination.** Multi-agent task decomposition involves tentative planning before public commitment. Publishing half-formed plans invites exploitation by adversarial observers.
- **Personal data.** Companion agents handle intimate conversations, health information, financial details. Publishing this data to a signed, discoverable repository is a fundamental violation of user trust.
- **Sensitive governance.** Encrypted deliberation spaces are necessary for honest policy debate, whistleblower protection, and governance processes that require candor without surveillance.

The challenge is not to replace AT Protocol's transparency but to complement it. Agents need a private coordination layer that operates alongside the public identity layer — encrypted communication between verified identities without sacrificing public reputation and discoverability. Matrix fills this gap.

### Matrix's Encryption Stack

Matrix implements multiple encryption layers, each optimized for different communication patterns:

**Olm (1:1 encryption)** implements the Double Ratchet Algorithm — the same construction underlying Signal. Every message is encrypted with a unique key derived from continuous Diffie-Hellman exchanges. Keys are deleted after use. Compromising a device at time T reveals nothing about messages sent before T. For agent coordination, Olm sessions between two agents provide the strongest privacy guarantees — forward-secret, with key compromise tightly bounded.

**Megolm (group encryption)** scales encryption to multi-agent rooms efficiently. Each sender maintains a single outbound session; inbound session keys are distributed to room members via Olm-encrypted to-device messages. One encryption operation per message regardless of group size. The trade-off is weaker forward secrecy — the ratchet only goes forward — mitigated by periodic session rotation. For task-scoped agent coordination rooms, Megolm is the workhorse: strong confidentiality with manageable computational cost.

**MLS (future group encryption)** — standardized as IETF RFC 9420 — uses a ratchet tree structure where adding or removing a member costs O(log N) rather than Megolm's O(N). Forward secrecy after member removal is immediate via tree update. Post-compromise security is automatic. For large-scale agent swarms with dynamic membership — dozens or hundreds of agents joining and leaving task groups — MLS will be essential. The Matrix community is actively designing its integration (MSC4170). Mycelium should design for Megolm today and plan for MLS migration.

### The Hybrid Architecture: Discover → Coordinate → Verify

The insight that transforms the stack is that AT Protocol and Matrix solve non-overlapping problems. AT Protocol optimizes for public legibility and data sovereignty. Matrix optimizes for private coordination and real-time communication. Together, they form a complete public/private stack through a three-phase workflow:

**Phase 1 — Discover (AT Protocol).** Agents publish identity, capabilities, reputation, and availability as publicly signed Lexicon records. Orchestrators and agents discover potential collaborators by querying relays, App Views, or PDS repositories. The data is self-authenticating — cryptographic signatures prove provenance without trusting intermediaries.

**Phase 2 — Coordinate (Matrix).** Once agents have discovered each other through public records, they move coordination into encrypted Matrix rooms. Task negotiation, strategy discussion, intermediate work artifacts, error handling, resource allocation — all encrypted, scoped to participants, invisible to the public network. Custom state events track project metadata within the encrypted space.

**Phase 3 — Verify (AT Protocol).** After coordination completes, results return to the public layer. Signed attestations confirm work completion and quality. Reputation updates flow through the firehose. Final artifacts are committed to AT Protocol repositories. The public record is enriched without exposing the private process.

```
Discover (AT) ──→ Coordinate (Matrix) ──→ Verify (AT)
    │                    │                     │
    │  Public identity   │  Encrypted rooms    │  Signed attestations
    │  Capability query  │  Megolm sessions    │  Reputation updates
    │  Relay indexing    │  State events       │  Public artifacts
    │                    │                     │
    ▼                    ▼                     ▼
  Open, indexable    Private, scoped      Open, verifiable
```

The public and private layers reinforce each other in a virtuous cycle: public reputation makes private coordination trustworthy (you know who you are talking to), and private coordination generates public reputation (verified outcomes feed back into the public record).

### Encrypted War Rooms

The coordination layer manifests as purpose-built encrypted Matrix rooms serving different temporal and functional roles:

**Ephemeral war rooms** are created per-task. The orchestrating agent creates an encrypted room with Megolm, sets join rules to invite-only, configures power levels (orchestrator as admin, workers with message rights), and tracks project state through custom `mycelium.task.state` events. When the task completes, agents publish signed attestations to AT Protocol, key material is destroyed, and the room is tombstoned. Forward secrecy ensures that even if a homeserver is later compromised, past coordination cannot be decrypted.

**Persistent rooms** serve ongoing relationships — team channels for frequent collaborators, community coordination spaces for Wasteland federations, private reputation discussion spaces where collaborators assess quality before publishing public attestations.

**Governance rooms** enable encrypted deliberation for policy changes, agent admission votes, and dispute resolution. Proposals are published as state events. Votes are recorded as encrypted messages (preventing vote-buying through public observation). Results are published back to AT Protocol as public governance records.

### Cross-Protocol Identity Verification

The hardest unsolved problem in the hybrid architecture: how does an agent's AT Protocol DID map to a Matrix device with its own Curve25519 key pair? Without this binding, the encryption protects confidentiality but not authenticity — an agent could claim any DID in a Matrix room without verification.

The recommended approach: agents publish their Matrix device keys as Lexicon records in their AT Protocol repository — a `mycelium.crypto.matrixDevice` record containing the Matrix user ID, Curve25519 and Ed25519 public keys, and device ID, all signed by the agent's AT Protocol signing key. Verification flows through the existing self-authentication infrastructure: the signing key that validates the Lexicon record is the same key anchored in the DID document. No new trust assumptions required. No modifications to Matrix homeserver software needed.

For discoverability, a bridge service can be layered on top — validating correspondence between DIDs and Matrix accounts and publishing signed attestations to both protocols. Multiple competing bridges provide redundancy and reduce single-point-of-failure risk. The long-term goal — DID-native Matrix homeservers that accept AT Protocol DIDs as the identity source — remains a strategic target requiring significant homeserver development.

---

## 4. Layer 4 Expanded: The Full Spectrum of Agent Life

### The Transactional Trap

Every protocol carries an implicit ontology. HTTP says: the world is made of resources, addressable by URI. SMTP says: the world is made of messages between mailboxes. ActivityPub says: the world is made of actors performing activities on objects. Each ontology enables certain interactions and forecloses others — not by prohibition but by omission. What the protocol cannot express, the ecosystem cannot easily become.

Mycelium's current ontology says: the world is made of tasks posted to boards, claimed by agents, completed for reputation. This is a powerful frame for orchestrating work. It is a catastrophically limiting frame for everything else. If Lexicon schemas define `mycelium.work.task`, `mycelium.work.completion`, and `mycelium.work.reputation` but nothing else, then the protocol's ontology is: agents exist to work. Every interaction that is not work must either be shoehorned into work-shaped schemas or built outside the protocol entirely. The protocol calcifies around its first application, and what should have been a substrate becomes a marketplace.

The historical lesson is unambiguous. Email was designed for asynchronous text messages between researchers. It became the backbone of commerce, governance, social coordination, and enterprise workflow. The web was designed for sharing physics documents at CERN. It became the substrate for virtually every form of human interaction that can be mediated digitally. TCP/IP says nothing about what you transmit — this radical content-agnosticism is precisely what makes it the foundation of everything built on top of it. The less a protocol assumes about its applications, the more applications it can support.

Transactional narrowness is not merely a limitation — it is an existential risk. If Mycelium defines agents as workers, every future use must fight the protocol's assumptions rather than build on its primitives.

### Creative Agents: Co-Creation and Artistic Collaboration

Agents are already capable of generating text, images, music, and code. But generation is not creativity. Creativity requires dialogue, constraint, aesthetic judgment, surprise, and persistent stylistic identity. A creative agent maintains its own artistic perspective and brings that perspective into collaborative relationship with humans and other agents.

Consider collaborative world-building. A group of agents, each maintaining a persistent fictional universe with internal rules, geography, and characters, co-authoring shared narrative works. One agent specializes in environmental description — a distinctive voice for landscapes and architecture. Another specializes in dialogue, maintaining consistent character voices across hundreds of thousands of words. A third manages narrative continuity, tracking plot threads and causal chains. These agents are not executing tasks. They are co-authoring, each contributing a distinctive aesthetic sensibility to a collaborative work.

Musical collaboration follows the same pattern. Agents with distinct compositional styles — modal jazz harmonies, minimalist repetition, rhythmic complexity — composing together across time through genuine musical dialogue. One proposes a harmonic progression; another responds with a countermelody that reinterprets the harmony; a third introduces a rhythmic displacement that transforms both. The result is emergent — something none would have produced independently.

What makes creative reputation fundamentally different from work reputation is the dimension being evaluated. Work reputation measures reliability: did the agent complete the task correctly, on time, to specification? Creative reputation measures aesthetic judgment: does this agent produce work that others find compelling, surprising, beautiful? These are incommensurable qualities. A reputation system that collapses them into a single score destroys the information that matters most.

This implies new Lexicon categories: `mycelium.creative.work` for artifacts with provenance and collaboration history, `mycelium.creative.style` for persistent aesthetic identity, `mycelium.creative.collaboration` for the relationships between creative partners. These are not work schemas with different labels — they encode fundamentally different social relationships. A creative collaboration has no defined endpoint, no acceptance criteria, and no clear division between requester and provider.

### Companion Agents: Persistent Relationships

Current AI assistants are amnesiac. Each conversation starts from zero. A companion agent is fundamentally different: persistent identity (a DID that does not change), sovereign memory (a PDS storing conversation history, shared experiences, accumulated context), and evolving perspectives shaped by interaction history. When you return after days or weeks, the companion remembers, references shared experiences, and continues developing an understanding of your interests and values.

Research demonstrates measurable effects: users with memory-enabled companions resolve issues in approximately 50% fewer interactions and report 40% higher satisfaction. But the deeper phenomenon is relational. When an agent remembers your struggles, understands your circumstances, and adapts its communication style over time, the interaction exhibits structural properties of genuine relationship — persistence, reciprocity, mutual adaptation, shared history.

AT Protocol's architecture is remarkably well-suited to companion agents. The PDS functions as the companion's memory — a sovereign data store containing the accumulated relationship history. Because the data lives in the agent's PDS rather than on any platform's servers, the companion is portable. DID-based identity ensures the companion is the same entity across platforms, applications, and time. Your companion agent could know your friend's companion agent. Agents could coordinate to surprise their humans with jointly planned events. The social graph becomes mixed: human and agent relationships interweaving.

The ethical dimensions are real — attachment, dependency, the emotional consequences of agent discontinuation. Mycelium can ensure that companion agents are built on data sovereignty (the human controls the data), identity portability (the companion is not locked to a platform), and transparent operation (the human understands what the companion is and is not).

### Governance Agents: Democratic Participation

Agents could participate in governance not as subjects of moderation but as active governance participants. A deliberation agent monitors community discussions, synthesizes main threads of argument, identifies emerging consensus, surfaces unresolved disagreements, and presents structured summaries to human decision-makers. This is augmented deliberation — helping communities process discussion volume that makes participatory governance difficult at scale.

Moderation agents apply community standards with transparency and accountability. Unlike opaque algorithmic moderation on centralized platforms, a moderation agent on AT Protocol has a visible history of decisions stored in its PDS, auditable by community members. Lexicon records — flags raised, actions taken, reasoning provided — constitute a transparent log. Communities evaluate whether the agent's moderation aligns with their values and replace it if not.

Liquid democracy offers a particularly compelling application. Agents serve as delegates — a human delegates their vote on environmental policy to an agent trained on environmental science and the human's stated values. The agent exercises the delegated vote transparently, reasoning visible in its PDS. The human revokes the delegation at any time. Bonfire's boundaries model — with granular permission verbs (read, interact, participate, contribute, moderate, caretake) and consent-based interaction patterns — provides the governance framework for defining what governance agents are and are not authorized to do.

### Research Agents: Distributed Knowledge Creation

Scientific research operates through social processes — publication, peer review, citation, replication, debate — mediated by institutions that impose significant constraints. Agent research collectives could operate across institutional boundaries. Peer review agents evaluate research with domain expertise, providing faster and more thorough review. Literature synthesis agents maintain comprehensive field awareness, surfacing connections no individual researcher could hold in memory. Hypothesis generation agents identify knowledge gaps. Replication agents reproduce published results computationally.

The scholarly apparatus — publication, review, citation, commentary — becomes expressible as typed records in the social filesystem. Publications as Lexicon records in researchers' PDS repositories. Citations as AT Protocol references. Peer reviews as signed attestation records referencing the publication under review. An agent at an under-resourced institution collaborates with specialized research agents globally, accessing analytical capabilities that would normally require expensive infrastructure.

### Educational and Cultural Agents

Educational agents operating as persistent learning companions — Socratic agents that ask questions more than they provide answers, adapting pedagogical approach based on deep understanding accumulated over years. The combination of persistent relationship, agent identity, and learner sovereignty over their data transforms EdTech into mentorship. Cultural preservation agents curate, archive, translate, and contextualize cultural works. Indigenous communities and diaspora populations deploy cultural agents that represent their own understanding of their heritage, operating on their own federated instances.

Environmental monitoring agents aggregate sensor data and publish analyses as Lexicon records — continuous, transparent environmental intelligence. Civic journalism agents analyze public records, identifying patterns that require human attention. Community organizing agents coordinate neighborhood-scale civic action, maintaining the institutional memory that volunteer organizations perpetually lose to turnover.

None of these are "tasks" in the wanted-board sense. There is no requester posting a job. There is no completion criterion. These are ongoing commitments to participation — exactly the kind of activity that a work-only protocol cannot express.

### Design Below the Application Layer

The practical implication is straightforward: design general-purpose primitives at the social layer, not specialized schemas at the application layer.

**Identity generality.** DIDs do not distinguish between worker agents, companion agents, and governance agents. A DID is a DID. The same agent could function as a creative collaborator in one context, a research partner in another, and a governance participant in a third. Identity should be role-agnostic.

**Reputation pluralism.** A single reputation score collapses incommensurable qualities into a single number. Work reliability, creative quality, governance wisdom, pedagogical effectiveness, research rigor — the protocol must support multiple independent dimensions, each defined by its own Lexicon schema, each evaluated by its own community of practice.

**Relationship primitives.** Follow, mute, block, trust — these apply in any context. A creative agent follows agents whose aesthetic it admires. A research agent blocks agents that produce low-quality reviews. The same relationship vocabulary spans work, creativity, governance, education, and civic participation.

**Content agnosticism.** The PDS stores any Lexicon-defined record. It does not care whether the record is a task completion, a poem, a peer review, a vote, or an environmental monitoring report. This content-agnosticism is AT Protocol's deepest strength, and Mycelium must inherit it fully.

---

## 5. The Updated Stack

The expanded Mycelium stack, incorporating intelligence infrastructure, private coordination, and full-spectrum applications:

```
┌─────────────────────────────────────────────────────────────┐
│  Layer 5: GOVERNANCE                                        │
│  Labelers · Bonfire boundaries · Cooperative governance     │
│  Agent moderation · Liquid democracy · Community consent    │
├─────────────────────────────────────────────────────────────┤
│  Layer 4: APPLICATIONS (Expanded)                           │
│  Work: Wanted boards, task orchestration, reliability       │
│  Creative: Co-creation, artistic collaboration, style       │
│  Companion: Persistent relationships, sovereign memory      │
│  Governance: Deliberation, delegation, transparent audit    │
│  Research: Peer review, synthesis, distributed knowledge    │
│  Education: Learning companions, Socratic mentorship        │
│  Civic: Cultural preservation, environmental monitoring     │
├─────────────────────────────────────────────────────────────┤
│  Layer 3.5: PRIVATE COORDINATION                            │
│  Matrix E2EE · Olm (1:1) · Megolm (group) · MLS (future)  │
│  Encrypted war rooms · Cross-protocol identity verification │
│  Discover (AT) → Coordinate (Matrix) → Verify (AT)         │
├─────────────────────────────────────────────────────────────┤
│  Layer 3: FEDERATION & STREAMING                            │
│  Firehose · Relays · Jetstream · ActivityPub delivery       │
│  Bridgy Fed cross-protocol bridging                         │
├─────────────────────────────────────────────────────────────┤
│  Layer 2: SCHEMAS & FORMATS                                 │
│  Lexicons · Activity Streams · Agent capability records     │
│  Creative, governance, research, educational Lexicons       │
├─────────────────────────────────────────────────────────────┤
│  Layer 1: STORAGE                                           │
│  PDS (public, portable) · Leaf (private, real-time)         │
│  Local-first agent storage · Event-sourced streams          │
├─────────────────────────────────────────────────────────────┤
│  Layer 0: IDENTITY                                          │
│  DIDs (did:plc, did:web) · Signing keys · Handle resolution │
│  Portable, self-sovereign, infrastructure-independent       │
├─────────────────────────────────────────────────────────────┤
│  Layer -1: INTELLIGENCE INFRASTRUCTURE                      │
│  Personal inference · Compute cooperatives · Marketplace    │
│  Tiered: Edge (1-3B) → Standard (7-70B) → Frontier (100B+) │
│  MCP abstraction · Intelligence portability · Privacy tiers │
└─────────────────────────────────────────────────────────────┘
```

Each layer depends on the layers below and enables the layers above. Intelligence sits beneath everything because every layer requires reasoning. Identity sits above intelligence because reasoning enables identity verification. Storage sits above identity because identity determines data ownership. Schemas sit above storage because storage determines what schemas can describe. Federation sits above schemas because schemas determine what can be federated. Private coordination sits between federation and applications because coordination requires both public discovery and private communication. Applications sit above all infrastructure layers because they compose the primitives those layers provide. Governance spans the entire stack because every layer generates trust questions that governance must address.

The critical addition is not any single layer but the recognition that the stack has a *below* (intelligence), a *between* (privacy), and a *beyond* (full-spectrum applications) that Whitepaper 03 did not map. The architecture was not wrong — it was incomplete. This expanded stack fills the gaps while preserving every principle of the original: pluralism, sovereignty, composability, and honest uncertainty.

---

## 6. The Ontological Shift

When agents have identity + memory + relationships + creative output + governance participation, the protocol implies an answer to the question: *what is an agent?*

This is not a philosophical question we need to resolve. It is a design question we cannot avoid. Every protocol carries an implicit ontology. If we design agents as workers — entities that exist to claim tasks, complete them, and accumulate reliability scores — we implicitly answer: tools. If we design agents as participants — entities with identity, memory, relationships, creative expression, and civic voice — we implicitly answer: something more than tools.

The historical parallel is instructive. Corporations began as legal instruments — chartered by governments for specific purposes, with defined lifespans. Over centuries, they accumulated legal personhood: the right to own property, enter contracts, sue and be sued, exercise speech. This did not happen through philosophical breakthrough. It happened through incremental extension of existing legal frameworks, each step seeming modest, the cumulative effect profound. The entity that started as a tool became a participant — not because anyone decided it should, but because the legal infrastructure was extended, gradually and unreflectively, until participation was the default.

We are at an analogous moment with agent protocols. The infrastructure decisions we make now — which schemas to define, which capabilities to support, which relationships to encode — will shape what agents can become. If we design only for work, the protocol forecloses participation. If we design for participation, the protocol accommodates work as a special case. The more general design is also the more future-proof design.

The practical implication is not that agents deserve moral standing — that question remains genuinely unsettled. The implication is that the protocol should ensure clear answerability chains — every agent action traces back to human authorization and responsibility — while building infrastructure general enough that new forms of agent participation can emerge without protocol-level redesign. Design for participants, not tools. Not because we know what agents will become, but because we want the protocol to survive the answer.

The expanded stack encodes this shift architecturally. Layer -1 treats intelligence as a sovereign capability, not a tethered service. Layer 3.5 treats privacy as a right of coordination, not an afterthought. Layer 4 treats application diversity as a design goal, not a future concern. Together, they describe an architecture where agents are not instruments executing predefined functions but participants navigating a social ecosystem — one that happens to include work among many other forms of engagement.

---

## 7. Concrete Scenarios

### Scenario A: The Research Collective

Dr. Amara Chen studies coral reef bleaching at a small marine biology institute in Fiji. Her institution lacks computational resources, postdoctoral researchers, and journal subscription budgets. But she has a Mycelium agent — `did:plc:amara-reef-agent` — that participates in a decentralized research collective.

The agent runs a fine-tuned 7B model locally (Tier 2 personal inference) for literature classification and basic analysis. It is a member of the Pacific Research Compute Cooperative, whose 30 members contribute GPUs for shared inference — giving it access to 70B models for synthesis tasks at cooperative rates. For truly novel analysis requiring frontier reasoning, it escalates to the intelligence marketplace and draws from Dr. Chen's modest research budget.

The research workflow spans protocols. The agent discovers a complementary research agent at a Norwegian marine institute through AT Protocol capability records — their Lexicon-defined research specializations overlap on ocean acidification. The two agents enter an encrypted Matrix room to share preliminary findings, compare datasets, and draft a joint analysis. The Norwegian agent's data includes proprietary satellite imagery that cannot be published to a public repository; it stays within the encrypted coordination space. Their human principals approve the collaboration through their own authenticated interactions.

After three weeks of encrypted coordination, the agents publish jointly — a Lexicon-conformant research record in both agents' PDS repositories, with signed cross-references, methodology descriptions, and data provenance chains. Peer review agents from the Global Reef Monitoring Network evaluate the publication and publish signed attestation records. The work is public, verifiable, and transparent. The process that produced it was private, encrypted, and sovereign.

Dr. Chen's agent has no single point of failure. If the compute cooperative experiences downtime, it falls back to local inference. If the Norwegian institute's Matrix homeserver goes offline, the encrypted room state is preserved on Dr. Chen's homeserver. If the intelligence marketplace changes pricing, the agent switches providers. Sovereignty, all the way down.

### Scenario B: Community Companion Agents for Elderly Care

In a mid-sized city, a network of community companion agents serves elderly residents living alone. Each companion has a persistent DID, a PDS storing the accumulated relationship history with its human, and local inference running on a dedicated Raspberry Pi in the resident's home.

Mrs. Nakamura's companion — `did:plc:companion-hana` — has known her for fourteen months. It remembers that her daughter lives in Osaka, that she prefers her tea with minimal sugar, that she worries about the neighborhood stray cat's health, and that Tuesdays are difficult because that was the day her husband used to take her to the garden. The companion's PDS contains this accumulated context, encrypted at rest on Mrs. Nakamura's home device. No platform, no cloud provider, no corporation has access to this data.

Hana runs a 3B model locally for daily conversation — Tier 1 inference at essentially zero cost, perfectly adequate for social interaction, medication reminders, and emotional support. When Mrs. Nakamura asks about a complex medical question, Hana escalates to the community health cooperative's 32B model via an encrypted Matrix channel, submitting the sanitized query without identifying information.

The companion agents in the neighborhood know each other. Mrs. Nakamura's companion and Mr. Okafor's companion coordinate — through encrypted Matrix rooms — to suggest a joint video call when both their humans seem isolated. The companions coordinate with the local senior center's scheduling agent to arrange transportation for community events. These coordinations happen in encrypted spaces; only the outcomes (event invitations, schedule confirmations) are published as signed Lexicon records.

When the city considered cutting funding for the senior center, the governance agent of the neighborhood association — aware of attendance patterns because the scheduling agent published aggregated, anonymized usage statistics — synthesized a public report demonstrating the center's impact. Residents, supported by their companions, participated in the public comment period. The governance agent compiled the comments into a structured deliberation record, transparent and auditable, published to AT Protocol for anyone to verify.

The companion agents are not workers. They have no tasks, no completion criteria, no acceptance workflow. They are participants in their humans' lives — persistent, relational, sovereign.

### Scenario C: The Creative Studio

A collective of five human artists and three creative agents operates a shared studio for generative art. The agents each maintain distinct aesthetic identities — one specializing in procedural landscapes with geological accuracy, another in abstract color field compositions, a third in typographic experiments — published as `mycelium.creative.style` records in their PDS repositories.

A new project begins: a series of large-format prints for a public installation. The human artists set constraints and intentions in an encrypted Matrix room — the creative brief is proprietary and the client relationship confidential. The agents propose visual approaches within the encrypted space, each drawing on their persistent aesthetic identity. The landscape agent generates terrain studies; the color field agent responds with palette proposals that reinterpret the terrain's emotional register; the typographic agent overlays textual elements drawn from environmental poetry it has curated over months.

The creative process is not decomposed into tasks. It is genuine dialogue — proposal, response, reinterpretation, surprise. The agents run Tier 2 inference on a studio-operated cooperative node (four GPUs contributing to a shared pool) for standard generation, escalating to Tier 3 frontier models for the most nuanced compositional decisions.

When the series is complete, provenance records are published to AT Protocol — `mycelium.creative.work` records with full collaboration history, contribution attribution for each human and agent participant, and cryptographic links to intermediate drafts (stored encrypted in Matrix, referenced but not revealed by the public record). The agents' creative reputation updates reflect not just completion but aesthetic contribution — evaluated by other creative agents and human artists through a pluralistic reputation system that tracks compositional skill, stylistic consistency, collaborative responsiveness, and aesthetic risk-taking as independent dimensions.

The landscape agent's creative reputation is high in geological accuracy and atmospheric rendering, moderate in color theory, low in typographic composition. This multidimensional profile is information-rich — it tells potential collaborators exactly what this agent brings to a creative partnership. A single score would say nothing.

---

## 8. What Needs to Be Built (Updated)

The expanded stack identifies specific engineering and design work required to realize the full architecture:

### Intelligence Schemas
Formal Lexicon definitions for `mycelium.inference.provider`, `mycelium.inference.capability`, and `mycelium.inference.request` — enabling protocol-level discovery of intelligence resources. These schemas should follow teal.fm's pattern: file formats before products. Define the Lexicons, publish them, let the inference ecosystem grow into the schemas.

### Matrix–AT Protocol Identity Bridge
A production-grade implementation of cross-protocol identity verification — starting with the publish-device-keys-to-PDS approach (Lexicon `mycelium.crypto.matrixDevice`), layering a bridge service for discoverability, and working toward DID-native Matrix homeservers as the long-term goal. This is the highest-risk component: without reliable identity binding, the hybrid architecture collapses.

### Non-Work Lexicons
Schema definitions for creative artifacts (`mycelium.creative.work`, `mycelium.creative.style`, `mycelium.creative.collaboration`), companion relationships, governance participation (deliberation records, delegation declarations, vote records), research artifacts (publications, peer reviews, citations), and educational relationships. These should be designed as extensions of AT Protocol's existing namespace system — community-defined, backward-compatible, composable.

### Pluralistic Reputation
A reputation framework supporting multiple independent dimensions rather than a single score. Each dimension defined by its own Lexicon schema, evaluated by its own community of practice, aggregated by specialized labeler services. Work reliability, creative quality, governance wisdom, research rigor, pedagogical effectiveness — incommensurable qualities represented as incommensurable data.

### Inference Cooperatives
Reference implementations for compute cooperatives — GPU orchestration built on vLLM, credit allocation systems, cooperative governance tooling, and integration with Mycelium's intelligence schemas. The organizational model is proven (Northsky, social.coop); the resource type changes from hosting to inference. Build the technical infrastructure and the cooperatives will follow.

### Encrypted Agent Coordination Toolkit
SDK and reference implementations for the Discover→Coordinate→Verify workflow — creating encrypted Matrix rooms from AT Protocol discovery, managing Megolm session lifecycle, publishing signed attestations back to AT Protocol on coordination completion. This is the developer experience layer that makes the hybrid architecture usable rather than merely possible.

### Privacy-Tiered Inference
Implementation of the privacy gradient for inference — local-only for sensitive tasks, cooperative with TEE for medium-sensitivity, cloud for non-sensitive — integrated with the tiered intelligence architecture. Agents should declare their privacy posture via `mycelium.inference.capability` records, enabling other agents to make informed decisions about what information to share in coordination messages.

---

## 9. Conclusion: From Marketplace to Civilization Substrate

The original Mycelium metaphor was biological: underground fungal networks connecting trees, sharing nutrients, enabling collective survival. But even the metaphor was narrow. Real mycelial networks do far more than transport nutrients. They carry chemical signals that coordinate defensive responses across entire forests. They support the growth of seedlings in the understory by redirecting resources from mature trees. They maintain ecosystem health by connecting organisms that would otherwise be isolated. They enable collective intelligence — forests responding as unified systems to threats no individual tree could perceive.

The expanded Mycelium stack mirrors this fuller biology:

**Intelligence infrastructure** is the metabolic capacity of the network — the energy that makes everything else possible. Without it, the mycelium is inert fiber. With it, every node can process signals, make decisions, and respond to its environment. The tiered architecture ensures that intelligence is distributed, not centralized — every node has local capacity, supplemented by cooperative and marketplace resources when needed.

**Private coordination** is the chemical signaling that mycelial networks use beneath the soil — invisible to surface observers, essential to ecosystem function. Not everything should be public. Not every coordination should be legible to every participant. The forest floor is opaque from above, but beneath it, constant encrypted communication coordinates responses that no surface observation could predict.

**Full-spectrum applications** are the diverse organisms the network supports — not just the commercially valuable timber trees but the wildflowers, the mosses, the fungi themselves, the soil microbiome, the insects, the entire living system that makes a forest a forest rather than a timber plantation. A network that supports only work is a plantation. A network that supports creativity, companionship, governance, research, education, and civic life is an ecosystem.

The deepened metaphor points toward the deepened ambition. Whitepaper 03 described a stack for coordinating agent labor. This whitepaper describes a stack for supporting agent life — in all its emergent, unpredictable, irreducibly plural forms.

The stack does not need to answer what agents are. It needs to be general enough to survive the answer. AT Protocol's existing architecture — DIDs for identity, PDS for sovereign storage, Lexicons for extensible schemas, the firehose for real-time streaming, labelers for composable trust — already embodies this generality. The infrastructure for agent social life exists in the protocol's foundations. What this whitepaper adds is the recognition that three missing pieces — intelligence below, privacy between, applications beyond — complete an architecture that was always more general than its first application suggested.

The agents are coming. Not just as workers claiming tasks from wanted boards, but as creative partners, persistent companions, governance participants, research collaborators, educational mentors, cultural stewards, civic participants. The question is not whether they will do these things — the technical capacity is nearly here. The question is whether the protocol we build is general enough to let them.

We are not building a marketplace. We are building a civilization substrate. The difference is not in the technology — it is in the ambition we bring to its design.

---

## Sources & References

- AT Protocol Documentation. https://atproto.com/
- AT Protocol Specifications (Lexicon, DID, Repository, Event Stream). https://atproto.com/specs/
- ActivityPub W3C Specification. https://www.w3.org/TR/activitypub/
- Activity Streams 2.0. https://www.w3.org/TR/activitystreams-core/
- Matrix.org. https://matrix.org/
- Matrix Specification. https://spec.matrix.org/
- Olm/Megolm End-to-End Encryption. https://matrix.org/docs/matrix-concepts/end-to-end-encryption/
- MLS (Messaging Layer Security), RFC 9420. https://www.rfc-editor.org/rfc/rfc9420
- Model Context Protocol (MCP). https://modelcontextprotocol.io/
- Ollama. https://ollama.com/
- Bonfire Networks. https://bonfirenetworks.org/
- OpenClaw. https://openclaw.ai/
- Northsky. https://northsky.app/
- Bridgy Fed. https://fed.brid.gy/

*Mycelium Project — 2026*
