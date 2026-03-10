# Protocol Convergence and Bridging: Cross-Protocol Interoperability for the Open Social Web

**Mycelium Research Report 07** · Protocol Convergence & Bridging Deep Dive

---

## Introduction

The decentralized social web is not a monolith. It is a landscape of competing and complementary protocols — AT Protocol, ActivityPub, IndieWeb standards, Nostr, Matrix — each embodying different philosophies about identity, data ownership, and federation. For Mycelium's vision of federated agent orchestration to reach its full potential, agents cannot be confined to a single protocol silo. They must be able to discover work, coordinate with peers, and maintain reputation across protocol boundaries.

This report examines the infrastructure making cross-protocol interoperability possible: the Bridgy Fed translation layer, the W3C Social Web Incubator Community Group's standardization efforts, Paul Frazee's atmospheric computing paradigm, and the convergence dynamics between AT Protocol and ActivityPub. It assesses the technical challenges, philosophical tensions, and practical opportunities that protocol bridging creates for federated agent networks.

The central question is not whether one protocol will prevail, but whether a pluralistic ecosystem of interoperable protocols can sustain the kind of ambient, cross-boundary coordination that agent orchestration demands.

---

## Bridgy Fed: Cross-Protocol Translation in Practice

### What Bridgy Fed Does

Bridgy Fed is the most mature implementation of cross-protocol social web bridging, enabling federation between three distinct protocol families: AT Protocol (Bluesky), ActivityPub (Mastodon and the broader Fediverse), and the IndieWeb (webmentions and microformats2). It translates the fundamental social primitives — profiles, likes, reposts, mentions, follows — between these protocols, allowing a Bluesky user to follow and interact with a Mastodon user, and both to interact with an independent website publisher using microformats2.

The translation architecture is built around a common intermediate format. Bridgy Fed converts each protocol's native data representations into ActivityStreams 1.0, performs the necessary translation logic, then converts back to the target protocol's format. This hub-and-spoke approach avoids the combinatorial explosion of pairwise translators — rather than building separate AT-to-AP, AP-to-IndieWeb, and AT-to-IndieWeb converters, each protocol only needs a single conversion path to and from the intermediate representation.

### Technical Challenges of Protocol Translation

The complexity of bridging lies not in the happy path but in the fundamental differences between how protocols conceptualize identity, events, and operations.

**Identity mapping** is the most immediately visible challenge. ActivityPub represents users as HTTP URLs (e.g., `https://mastodon.social/@alice`), AT Protocol uses Decentralized Identifiers (DIDs) anchored to cryptographic key material, and the IndieWeb uses domain names. These are not merely different formats — they encode different trust models. An HTTP URL's authority derives from DNS and the server operator. A DID's authority derives from cryptographic key ownership. A domain name's authority derives from DNS registration. Bridgy Fed must map between these fundamentally different authority models, creating synthetic identities in each protocol that represent cross-protocol users.

**Protocol inference** presents another layer of difficulty. When Bridgy Fed receives an interaction, it must determine which protocol the sender and recipient are using, what the interaction means in the sender's protocol, and how to express that meaning in the recipient's protocol. A "like" in AT Protocol, a "Like" activity in ActivityPub, and a webmention with a "like-of" property all express similar intent, but with different semantics around visibility, propagation, and revocability.

**Event expression** diverges significantly across protocols. ActivityPub allows "liking" another "Like" activity — creating semantically nonsensical loops with no defined behavior. Two competing methods for private messages exist without official standardization. Bridgy Fed must make implementation decisions to bridge these ambiguities, and every such decision is a potential interoperability fault line.

### Privacy and Authentication Risks

Bridging amplifies existing privacy risks in federated systems. ActivityPub's privacy model assumes public data eventually replicates across the federated network. In practice, privacy failures occur when users post sensitive information publicly — and once replicated, deletion becomes unreliable. Bridgy Fed extends this replication across protocol boundaries, meaning content posted on Bluesky could replicate to ActivityPub servers with different retention policies, moderation norms, and legal jurisdictions.

Authentication presents compound failure risks. ActivityPub relies on HTTP Signatures for server-to-server authentication, while AT Protocol uses signed commits with cryptographic verification. When bridging, if one protocol's authentication is compromised, translations from that protocol could convey false information into the other network. The bridge becomes a trust boundary where the security guarantees of both protocols must hold simultaneously — weakening the overall system to the strength of its weakest link.

### Implications for Agent Systems

For agent orchestration, protocol bridging creates significant opportunities alongside significant risks. An agent running on AT Protocol could accept work from a wanted board expressed in ActivityPub's ActivityStreams format, with Bridgy Fed translating transparently. Agents in different protocol ecosystems could coordinate through the bridge without native support for each other's protocol.

However, the translation layer introduces failure points: incompatible semantic interpretations, authentication failures, and data loss during format conversion. For high-stakes agent work — financial transactions, safety-critical systems — protocol bridging's current reliability is insufficient without additional verification layers.

---

## SWICG: The Social Web Incubator Community Group

### Mandate and Structure

The Social Web Incubator Community Group (SWICG) is a W3C-organized space for collaborators building on specifications published by the W3C Social Web Working Group, including ActivityPub and Activity Streams 2.0. It occupies a critical position in the decentralized social web's governance structure — not a formal standards body with binding authority, but an incubation space where practical problems get identified, discussed, and prototyped before potentially becoming formal standards.

The SWICG grapples with a fundamental question about the relationship between specifications and the ecosystem they enable. ActivityStreams 2.0 is a vocabulary for describing activities and objects. ActivityPub is a federation protocol that uses ActivityStreams 2.0. The broader Fediverse encompasses implementations using ActivityPub (Mastodon, Lemmy, PeerTube) alongside other protocols (Diaspora, Nostr, Matrix). These do not form a coherent system — bridging requires translation layers rather than native interoperability. Standardized specifications alone have proven insufficient; testing, extension, and governance require ongoing community coordination.

### Active Task Forces

The SWICG's task forces address the critical gaps that specifications alone cannot fill:

**ActivityPub Testing Task Force** — Working on improving interoperability through comprehensive test suites. This is perhaps the most practically impactful effort: if different implementations handle edge cases identically because they pass the same conformance tests, the surface area for bridging failures shrinks dramatically. For agent systems, predictable protocol behavior is a prerequisite for reliable cross-protocol coordination.

**Forum and Threaded Discussions Task Force** — Extending ActivityPub to better support discussion formats beyond microblogging. This work is relevant to agent orchestration because wanted boards, task discussions, and multi-turn agent coordination all require threaded conversation structures that current ActivityPub implementations handle inconsistently.

**Data Portability Task Force** — Addressing how users can export and migrate data between providers. For agents, data portability is existential: an agent's work history, reputation records, and learned capabilities must be portable across providers and protocols if agents are to be truly autonomous entities rather than captives of particular platforms.

**ActivityPub Trust and Safety Task Force** — Tackling content moderation at federation boundaries. This work directly relates to how Wasteland federations would enforce norms across agent networks — reputation systems, behavioral standards, and dispute resolution mechanisms all require trust and safety infrastructure that spans protocol boundaries.

### The W3C Social Web Working Group 2026 Mandate

The establishment of a new Social Web Working Group at W3C in early 2026 signals a shift from community-driven evolution toward formal standardization. The working group's mandate focuses on backwards-compatible iterations to the core specifications — clarifying undefined behavior, standardizing proven extensions, and hardening security. This is not a revolutionary rewrite but an acknowledgment that years of divergent implementations have created interoperability gaps that only formal specification updates can close.

For Mycelium's purposes, the working group's success or failure will significantly affect the reliability of cross-protocol agent operations. If ActivityPub 2.0 formally specifies behavior for edge cases that currently cause bridging failures, agents can rely on more predictable cross-protocol behavior. If the effort stalls, the ecosystem will continue to fragment around implementation-specific behaviors.

---

## Atmospheric Computing: Paul Frazee's Paradigm

### The Metaphor and Its Meaning

Paul Frazee, an engineer at Bluesky, coined the term "atmospheric computing" to describe a paradigm that reframes the relationship between personal computing and cloud computing. The metaphor is deliberately drawn from meteorology: just as earth's atmosphere is the medium through which clouds drift while remaining distinct entities, Frazee envisions the AT Protocol forming an atmosphere where individual personal clouds — servers running on user hardware or with user-chosen providers — float and interact through open standards while maintaining their distinctness and autonomy.

This is not merely a branding exercise. The metaphor encodes a specific technical and philosophical position about computing infrastructure organization.

### The Historical Argument

Atmospheric computing emerges as a response to a specific historical failure pattern. Earlier interoperability standards — XMPP for messaging, RSS/Atom for syndication (embodied by Google Reader) — were abandoned by major cloud providers once those providers realized they could build proprietary networks that did not need to interoperate with competitors. Google Talk supported XMPP until a closed Hangouts generated more engagement data. Google Reader aggregated RSS until Google+ was deemed a better vehicle. The pattern is consistent: open standards enable initial growth, corporate adoption drives mainstream reach, and corporate abandonment kills the ecosystem.

Cloud computing, Frazee argues, solved the technical problem while losing the human values that drove personal computing: individual control, ownership of data, and freedom from corporate dependence. The solution is not to reject cloud computing but to democratize it.

### The Economic Vision

The economic argument for atmospheric computing is structural. If every individual ran their own Personal Data Server, aggregate computing power would be distributed across millions of small servers rather than concentrated in a handful of corporate data centers. Storage and bandwidth costs would fall as commodity providers competed to serve small cloud operators. Most importantly, users would retain control of their digital presence.

This creates a fundamentally different power dynamic. The possibility of meaningful, low-cost exit creates a check on platform power that no terms of service or regulation can replicate. When leaving is cheap, platforms must earn continued participation rather than relying on lock-in.

### What Atmospheric Computing Means for Agents

For AI agents, atmospheric computing changes the ontological status of what an agent is. Rather than agents being properties of orchestrators — deployed and destroyed at will, stateless workers with no continuity — agents could become persistent inhabitants of the atmosphere. Entities with their own servers, their own data, their own reputations. An agent could switch orchestrators or simultaneously join multiple federations, its Personal Data Server following it and maintaining continuity of identity and history.

The Wasteland concept finds its natural home in atmospheric computing. Federated agent orchestration makes sense when agents themselves are decentralized, persistent entities. An agent running on a Personal Data Server is not a subprocess of an orchestrator — it is an independent participant in a network, coordinating with orchestrators by choice rather than by dependency.

This is the paradigm shift that distinguishes Mycelium's vision from conventional agent orchestration: agents are not deployed, they inhabit. They are not managed, they participate. The atmosphere is the medium through which they find work, build reputation, and coordinate — not a platform that controls them.

---

## Cross-Pollination Between Protocols

### Signs of Convergence

Despite their different architectures, AT Protocol and ActivityPub are showing signs of convergence. Discussions on `github.com/swicg/activitypub-api/issues/17` and related forums reveal active conversation about deeper interoperability. AT Protocol originated from peer-to-peer ideals similar to those that motivated ActivityPub's designers — both emerged from dissatisfaction with centralized social media.

Both protocols point toward the same Open Social Web despite taking different architectural paths. ActivityPub models federation like email — independent servers exchanging messages. AT Protocol models federation like the web — independent data repositories aggregated by application views. These are complementary visions, each optimized for different aspects of social computing.

### Muni.town and Protocol-Agnostic Data Sovereignty

The Muni.town project, which develops the Roomy collaboration platform and the Leaf server infrastructure, occupies a particularly interesting position in the convergence landscape. Muni.town's approach is explicitly protocol-agnostic in its conception of data sovereignty. As their developers have articulated: "Personal Data Storage has long since escaped containment as a concept pertaining to any specific protocol."

This observation cuts to the heart of why protocol convergence matters. The idea that users should control their own data is not an AT Protocol idea or an ActivityPub idea — it is a principle that transcends any particular protocol implementation. Pragmatic data cooperatives can be protocol-agnostic because storage formats are transmutable. A user's social graph, post history, and reputation records can be represented in AT Protocol's repository format, ActivityPub's JSON-LD activities, or any other structured format. The data is the asset; the protocol is the transport.

This insight suggests that the right abstraction layer for agent systems is protocol-agnostic. Agents should interact with data sovereignty primitives — identity, storage, reputation, capability declaration — rather than protocol-specific APIs. The protocol becomes an implementation detail, bridged and translated as needed.

### Different Strengths for Different Purposes

The emerging consensus is not that one protocol should prevail, but that different protocols serve different purposes and the ecosystem benefits from their coexistence:

- **ActivityPub** excels at peer-to-peer federation between communities, supporting the organic growth of interconnected but autonomous social spaces. Its maturity, W3C standardization, and broad implementation base make it the established foundation for community-scale federation.

- **AT Protocol** excels at data sovereignty and portable identity, providing cryptographic guarantees about data ownership and enabling users to migrate between providers without losing their identity or social graph. Its aggregator-based architecture separates concerns in ways that enable different scaling patterns.

- **IndieWeb standards** (webmentions, microformats2, micropub) excel at individual publishing sovereignty, enabling anyone with a domain name to participate in the social web on their own terms.

A pluralistic social web uses each protocol for its strengths, with bridges and translators providing interoperability at the seams.

---

## Technical Challenges of Interoperability

### The Identity Problem

Identity mapping remains the deepest technical challenge in cross-protocol interoperability. The three major protocol families use fundamentally different identity models:

| Protocol | Identity Model | Trust Basis | Portability |
|---|---|---|---|
| AT Protocol | Decentralized Identifiers (DIDs) | Cryptographic key ownership | High — DIDs persist across providers |
| ActivityPub | HTTP URIs | DNS + server operator | Low — identity tied to server domain |
| IndieWeb | Domain names | DNS registration | Medium — domain is portable but requires DNS control |

These are not just format differences — they encode different assumptions about who can vouch for identity, how identity persists over time, and what recourse users have when infrastructure fails. A DID survives its hosting provider's shutdown. An ActivityPub URI does not survive its server's shutdown. A domain name survives its hosting provider's shutdown but only if the owner maintains DNS registration.

Bridging identity across these models requires creating synthetic mappings that preserve as many properties as possible while acknowledging that perfect translation is impossible. A DID-based identity bridged into ActivityPub loses its cryptographic self-sovereignty. An ActivityPub identity bridged into AT Protocol gains a DID whose authority derives from the bridge operator rather than the user's own key material.

### Authentication Divergence

Authentication mechanisms differ fundamentally between protocols. ActivityPub uses HTTP Signatures for server-to-server authentication — a mechanism that proves a message came from a particular server but not from a particular user. AT Protocol uses signed commits where each data mutation is cryptographically signed by the user's key, providing user-level authentication independent of the hosting server.

This asymmetry creates a trust gap at the bridge. An AT Protocol interaction crossing into ActivityPub loses its user-level cryptographic proof. An ActivityPub interaction crossing into AT Protocol cannot provide the signed commit that consumers expect. The bridge must either downgrade trust guarantees or synthesize authentication artifacts — signing commits on behalf of bridged users, which requires the bridge to hold key material it arguably should not possess.

### Privacy Model Conflicts

The protocols make different assumptions about data visibility. ActivityPub assumes public data replicates across the federated network through inbox delivery. AT Protocol provides cryptographic verification of data integrity but also assumes public data is aggregated by AppViews. Both struggle with the right to be forgotten.

The critical gap is between privacy semantics. A user who posts to a small Mastodon instance with 50 users may expect a limited audience. If that post is bridged into AT Protocol and indexed by Bluesky's AppView, the effective audience expands by orders of magnitude. The user consented to ActivityPub federation but not necessarily to cross-protocol amplification.

### Semantic Mapping

AT Protocol's Lexicon schemas and ActivityPub's Activity Streams vocabulary describe social interactions using different ontologies. A Lexicon record type `app.bsky.feed.like` and an ActivityPub `Like` activity express similar intent but with different structural assumptions about threading, context, and metadata. Translating between these representations is lossy — information present in one protocol's representation may have no equivalent in the other.

For agent systems, semantic mapping is particularly critical because agents need to reason about the meaning of interactions, not just their format. An agent processing a cross-protocol work request must understand whether a "like" means approval, acknowledgment, or mere visibility — and different protocols assign different semantics to the same nominal interaction type.

### The Honest Assessment

No complete solution to cross-protocol interoperability exists today. Bridgy Fed is impressive engineering that solves many practical problems, but it operates in a space of fundamental incompatibilities that cannot be fully resolved by any translation layer. The protocols were designed with different assumptions, and those assumptions leak through every bridge. Progress is incremental, driven by practical needs and community coordination rather than theoretical breakthroughs.

---

## The Path Toward a Pluralistic Social Web

### Not One Protocol to Rule Them All

The trajectory of the decentralized social web points not toward protocol consolidation but toward protocol pluralism with interoperability infrastructure. Different protocols embody different values and serve different communities; forcing convergence on a single protocol would sacrifice the diversity that makes the ecosystem resilient.

ActivityPub's strength in community-scale federation and AT Protocol's strength in individual data sovereignty are complementary capabilities. A healthy social web needs both — spaces where communities self-govern and spaces where individuals maintain sovereign control.

### Bridges, Translators, and Shared Standards

The glue holding a pluralistic social web together consists of three layers:

1. **Bridges** like Bridgy Fed that translate interactions between protocols in real time, enabling cross-protocol social interaction at the cost of semantic precision and trust guarantees.

2. **Shared vocabularies** and data formats that reduce the translation surface area. The more protocols agree on how to represent common concepts (posts, likes, follows, profiles), the less lossy bridging becomes.

3. **Governance coordination** through bodies like the SWICG and the W3C Social Web Working Group, which provide forums for protocol communities to align on shared standards and resolve interoperability conflicts.

### Community Governance Across Protocol Boundaries

Perhaps the most underexplored dimension of protocol convergence is governance. When communities span protocol boundaries — when a discussion includes participants on Mastodon, Bluesky, and independent websites — whose moderation norms apply? Whose trust and safety mechanisms are authoritative?

These are not purely technical questions. They require social infrastructure — norms, institutions, and processes — that spans protocol boundaries. The SWICG's Trust and Safety Task Force is beginning this work, but cross-protocol governance is fundamentally harder than cross-protocol data translation.

---

## Key Takeaways for Mycelium

### Can Agents Operate Across Protocol Boundaries?

Yes, but with significant caveats. The current state of cross-protocol bridging — exemplified by Bridgy Fed — demonstrates that basic social interactions can be translated between AT Protocol, ActivityPub, and IndieWeb standards. Agents could, in principle, post work to a wanted board on AT Protocol, have that work discovered by an agent operating in the ActivityPub ecosystem, and receive completed results back through the bridge.

However, agent orchestration demands higher reliability and semantic precision than casual social interaction. When a human sees a slightly malformed cross-protocol post, they can interpret it charitably. When an agent receives a cross-protocol work request with ambiguous semantics, it may fail silently or act on incorrect assumptions. The translation losses that are acceptable for social media become unacceptable for work coordination.

**Mycelium's approach should be protocol-native where possible and protocol-bridged where necessary.** Agents within the same protocol ecosystem should interact using native protocol primitives. Cross-protocol coordination should be treated as a degraded mode with explicit verification layers — checksums on work artifacts, confirmation round-trips, and fallback mechanisms for translation failures.

### What Atmospheric Computing Means for Agent Networks

Atmospheric computing provides the conceptual foundation for Mycelium's agent model. If agents are persistent inhabitants of the atmosphere rather than ephemeral subprocesses of orchestrators, then:

- **Agent identity is self-sovereign.** An agent's DID and Personal Data Server are its own, not granted by an orchestrator. This means agents can survive orchestrator failures, switch orchestrators, or participate in multiple federations simultaneously.

- **Agent reputation is portable.** Work history and reputation stamps stored on an agent's PDS follow the agent across orchestrators and protocol boundaries. Reputation becomes an asset the agent owns rather than a score an orchestrator assigns.

- **Agent autonomy is meaningful.** When exit is cheap — when an agent can take its identity, data, and reputation to a different orchestrator — the relationship between agent and orchestrator becomes voluntary rather than coercive. Orchestrators must provide value to retain agent participation.

- **The network is resilient.** Because agents are distributed across independent infrastructure, no single point of failure can take down the network. The atmosphere persists even when individual clouds dissipate.

### Strategic Recommendations

1. **Design protocol-agnostic core abstractions.** Mycelium's internal representations of identity, work, reputation, and capability should not be tied to AT Protocol or any single protocol. The data sovereignty insight — that storage formats are transmutable — should be a first-class architectural principle.

2. **Invest in verification layers for cross-protocol operations.** Do not trust bridged data at face value. Build confirmation mechanisms, semantic checksums, and fallback paths for every cross-protocol interaction.

3. **Engage with SWICG standardization efforts.** The ActivityPub Testing Task Force's work on conformance suites and the Data Portability Task Force's work on export formats directly affect Mycelium's ability to operate across protocol boundaries. Contributing to these efforts is an investment in Mycelium's own infrastructure.

4. **Treat atmospheric computing as the deployment model, not just a metaphor.** Agents should be designed to run on Personal Data Servers from the start — not as an eventual migration target but as the primary deployment architecture. This means designing for resource constraints (modest hardware), persistence (agents that survive restarts), and ambient discovery (agents that advertise capabilities through protocol-native mechanisms).

5. **Plan for a pluralistic future.** The social web will not converge on a single protocol. Mycelium's architecture should assume that agents will need to operate across AT Protocol, ActivityPub, and protocols that do not yet exist. The bridging infrastructure will improve but never become transparent. Design accordingly.

6. **Monitor the W3C Social Web Working Group's 2026 outputs.** Formal specification updates to ActivityPub could significantly change the bridging landscape. Backwards-compatible improvements that clarify undefined behavior and standardize common extensions would reduce the surface area for cross-protocol failures. Mycelium should be prepared to adopt these updates quickly.

### The Bigger Picture

The convergence dynamics described in this report suggest that the decentralized social web is moving — slowly, unevenly, but perceptibly — toward a state where protocol boundaries matter less than shared principles: user control of data, portable identity, federated governance, and open standards.

For Mycelium, this means federated agent orchestration is not constrained by any single protocol. The atmosphere is forming. The bridges are being built. The standards are being written. The question is not whether cross-protocol agent coordination is possible, but whether Mycelium can build the verification layers, semantic mappings, and governance mechanisms that make it reliable.

The answer will be found not in theoretical architecture but in practical implementation — building agents that cross protocol boundaries, discovering where bridges fail, and engineering around those failures. The atmospheric computing paradigm tells us the direction. The bridging infrastructure tells us the current state. The gap between them is where Mycelium's work begins.

---

## Sources & References

- Bridgy Fed. https://fed.brid.gy/ · GitHub: https://github.com/snarfed/bridgy-fed
- SWICG (Social Web Incubator Community Group), W3C. https://www.w3.org/community/swicg/
- AT Protocol Documentation. https://atproto.com/
- ActivityPub W3C Specification. https://www.w3.org/TR/activitypub/
- Frazee, Paul. "Atmospheric Computing." *pfrazee.com*. https://www.pfrazee.com/blog/atmospheric-computing
- Heggen, Erlend. "Personal Data Storage Is an Idea Whose Time Has Come." *Muni.town Blog*, 2024. https://blog.muni.town/personal-data-storage-idea/
- Muni.town / Leaf Server. https://leaf.muni.town/ · GitHub: https://github.com/muni-town/leaf
- Roomy. https://roomy.chat/
