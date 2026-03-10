# AT Protocol Deep Dive: Architecture, Identity, and the Open Social Data Layer

> **Mycelium Research Series — Report 2**
> Investigating how federated agent orchestration can be built on decentralized social protocols.

---

## Overview & Philosophy

The AT Protocol — the **Authenticated Transfer Protocol** — is pronounced "at," as in the @ symbol. Created by the Bluesky team, it represents a fundamentally different approach to decentralized social networking compared to predecessors like ActivityPub. Where ActivityPub modeled itself after email (servers exchanging messages), AT Protocol models itself after the web: independent nodes *publish* data that aggregators *collect and index* into different views and applications.

The driving philosophy can be distilled into a single sentence from Dan Abramov: **"What open source did for code, open social does for data."** AT Protocol externalizes the internal architecture of large-scale backend systems — caches, replicas, stream processors, materialized views — as the protocol itself, distributing them across independent operators while ensuring they all operate over consistent underlying data.

This is not a peer-to-peer system. It is not a federation of equivalent servers. It is a modular, microservice-based design that *selectively* applies decentralization principles to create what amounts to an **inverted database architecture**:

- **Storage** is decentralized (Personal Data Servers)
- **Indexing** is delegated (Relays)
- **Application logic** is separated (App Views)
- **Moderation** is composable (Labelers)

Each layer can be operated by different entities, optimized independently, and swapped without disrupting the others. The user's data remains *theirs* — cryptographically signed, portable, and verifiable — regardless of which services they choose at each layer.

For the Mycelium project, this architecture is significant because it solves a problem we face directly: how do you build a system where autonomous agents can operate across organizational boundaries while users retain sovereignty over their data and identity?

---

## Core Architecture Components

### Personal Data Server (PDS)

The PDS sits at the foundation of the entire protocol. Each user has exactly one PDS that hosts their account data. It is the user's **home in the cloud** — their data authority.

**Responsibilities:**
- Hosts the user's data repository (posts, profiles, likes, follows, all records)
- Manages identity information (cryptographic keys, handle-to-DID mappings)
- Handles key management (credential rotation, account recovery)
- Provides an HTTP API (XRPC) for authenticated client requests
- Manages OAuth 2.0 authentication flows
- Stores binary data (images, videos) as blobs

**Implementation details:** The PDS uses per-user SQLite databases, keeping the storage model simple and portable. The `AppContext` class serves as the main entry point, wiring together services for account management, repository operations, and API routing. The computational requirements are intentionally low — a PDS only needs to serve its own hosted accounts, not maintain a global view of the network. This means even individuals running on modest hardware (a Raspberry Pi 5 with 1 CPU core and 1 GB of RAM has been demonstrated) can operate their own PDS.

**The critical insight:** Because the PDS doesn't need to index or aggregate the global network, self-hosting becomes practical. This stands in stark contrast to ActivityPub instances, which must handle complex synchronization logic as they federate with more servers.

**Data availability:** Even if a specific PDS goes offline, the user's data has already propagated through relays and remains accessible. Users can request their data export, migrate to a new PDS, and continue participating without losing their archive.

### App Views (Bsky)

If the PDS is where data *lives*, the App View is where data becomes *useful*. App Views consume the firehose from relays and transform raw repository data into user-facing experiences. Bluesky's App View implements the `app.bsky.*` API endpoints — the microblogging application layer.

**Internal pipeline:** Bluesky's App View follows a three-stage architecture:
1. **DataPlane** — Indexes events from the firehose into PostgreSQL databases
2. **Hydrator** — Enriches raw records with related data (author profiles, engagement counts, media)
3. **Views** — Assembles hydrated data into API responses for clients

**What App Views do:**
- Count likes, collate reply threads, maintain follower relationships
- Construct personalized feeds and deduplicate graph relationships
- Handle record-level and account-level deletions
- Track the most recent processed revision for each repository
- Fetch media blobs from PDSes, process them (resize, thumbnail), serve via CDN
- Store private user preferences, handle notifications, manage mute lists
- Subscribe to labelers and apply moderation rules
- Implement authentication (service auth tokens proxied through PDSes, or direct OAuth)

**The key design principle:** Different applications have fundamentally different indexing and aggregation needs. A microblogging app needs reverse-chronological timelines and like counts. A photo-sharing app needs grid layouts and EXIF metadata. By separating application logic from protocol logic, multiple App Views can coexist — each optimized for its use case, all operating from the same underlying data.

This separation enables **credible exit**: users can switch between competing App Views without losing data, followers, or posts. If you're dissatisfied with Bluesky's implementation, you could theoretically switch to an alternative App View while retaining your complete social graph.

### Relays

Relays are the indexing infrastructure that makes global data discovery possible. A relay crawls the network by continuously fetching repository updates from PDSes, aggregates them, and produces the **firehose** — a unified stream of all public network activity.

Relays are *technically* optional. Applications could query PDSes directly. In practice, relays dramatically simplify development: instead of maintaining connections to millions of individual PDSes, applications subscribe to a single relay and receive all updates in one stream.

**Scalability solutions:**
- **Rainbow**: A firehose fan-out service maintaining a single upstream relay connection and re-broadcasting to many clients, reducing bandwidth pressure
- **Sharding**: The protocol anticipates eventual sharding where different relays handle different network segments based on account identifiers

**The centralization tension:** Relays are the most criticized component because they represent a potential centralization point. As of early 2026, Bluesky operates the primary relay, and the costs of running production-grade relay infrastructure have limited independent alternatives. Relays also perform content filtering (removing CSAM, spam), introducing a gatekeeper role. The "Free Our Feeds" initiative has emerged specifically to develop relay infrastructure independent of Bluesky.

### Labelers

Labelers implement AT Protocol's moderation philosophy: **separate speech from reach**. Labels are metadata objects attached to accounts or records containing moderation decisions, content warnings, or contextual information.

**Architecture:**
- Labelers function as regular AT Protocol accounts with their own repositories
- They declare their role through a service declaration in their DID document
- They subscribe to network events (via firehose) and make independent moderation judgments
- Applications subscribe to labelers they trust and apply those labels when rendering content
- Users can customize which labelers they subscribe to and how each labeler's decisions are weighted

**Bluesky's implementation uses Ozone**, an event-sourced moderation platform. Ozone provides a web interface for manual moderation decisions and can leverage classifiers (including fine-tuned language models) for automated decisions through **Osprey**, an event stream decisions engine.

The system enables specialized moderation — one labeler could be strict about harassment, another permissive. Communities can operate labelers reflecting their values. The protocol includes sophisticated label definitions specifying whether content should be blurred, warning severity, and default behavior.

---

## Identity System

### Decentralized Identifiers (DIDs)

DIDs form the permanent, portable identity anchor for all AT Protocol accounts. They are W3C standard cryptographically verifiable identifiers that do not depend on any centralized registry.

**Two supported methods:**

| Method | Description | Strengths | Weaknesses |
|--------|-------------|-----------|------------|
| `did:plc` | Self-authenticating, AT Protocol-specific. Includes encoded cryptographic material. | Enables self-authentication and recovery without external registries. Rotation keys allow identity changes without PDS involvement. | Controlled by a PLC directory currently operated by Bluesky. |
| `did:web` | Tied to a domain name the user owns. Resolved via DNS queries. | Simple for users with domain registrations. Familiar DNS infrastructure. | If domain control is lost, DID document cannot be updated. |

### DID Documents

Both methods publish a DID Document containing:
- **Signing key**: The primary public key that validates the user's data repository and signs all records
- **PDS service endpoint**: Where the account's data is currently hosted
- **alsoKnownAs**: The user's handle(s)
- **Rotation keys** (did:plc only): Allow users to assert identity changes independently of their PDS — critical for account recovery and migration

### Handles

Handles are the human-readable layer — DNS hostnames that can be changed at any time without affecting the underlying DID or any protocol relationships.

**How handles work:**
- A user with domain `alice.example.com` creates a DNS TXT record pointing to their DID
- Changing handles is as simple as updating DNS records
- Followers continue following the same DID regardless of handle changes
- Multiple handles per account are supported

**DNS dependency:** Handles rely on external infrastructure — domain registrars, DNS providers, the DNS system itself. If a registrar seizes a domain or DNS becomes unavailable in a region, users lose handle-based discoverability (though their DID remains accessible to anyone who already has it).

### Account Portability

The dual identity system (permanent DID + mutable handle) enables AT Protocol's defining feature: **account portability**. Users can migrate their accounts to entirely different PDSes operated by different organizations. They update their DID document to point to the new PDS, and all future queries resolve to the new location. Social graph, followers, and posts remain associated with the permanent DID, not the hosting provider.

### Critical Limitation: DID Immutability

DIDs cannot be changed once created. A user who started with `did:plc` and later wants `did:web` cannot convert — they would need to create a new account, losing all followers and social connections. The protocol tracks relationships by DID, not by handle. This inflexibility is a significant constraint for long-term account management and has generated substantial community discussion.

---

## Lexicons — The Schema Language

Lexicons are AT Protocol's answer to interoperability. They are a JSON-based schema language — similar to JSON-Schema and OpenAPI but tailored specifically to AT Protocol's requirements. The crucial principle: **the protocol IS the API**. Lexicon schemas are the single source of truth for data formats, API behaviors, and event stream messages.

### Namespaced Identifiers (NSIDs)

Every Lexicon schema is assigned an NSID using reverse-DNS notation:

```
com.atproto.repo.getRecord    → Core protocol: get a record from a repository
app.bsky.feed.post             → Bluesky app: a microblogging post
app.bsky.actor.getProfile      → Bluesky app: fetch a user profile
com.example.photo.upload       → Hypothetical third-party: photo upload
```

**NSID governance:** Authority over a namespace is delegated to whoever controls the corresponding domain. If you own `example.com`, you can publish schemas in `com.example.*` without permission from any central authority. This enables rapid innovation while preventing namespace collisions.

### Schema Types

Lexicon schemas support four primary definition types:

| Type | Protocol Concept | Transport |
|------|-----------------|-----------|
| **Record** | Data stored in repositories | CBOR-encoded JSON in MST |
| **Query** | Read operations | HTTP GET via XRPC |
| **Procedure** | Write/action operations | HTTP POST via XRPC |
| **Subscription** | Live event streams | WebSocket |

Within these definitions, Lexicon supports primitives (boolean, integer, string), containers (arrays, objects), and protocol-specific types (blobs for binary files, CID-links for content-addressed references, refs for schema cross-references).

### Backward Compatibility Rules

This is a critical design constraint: once a Lexicon schema is published, **constraints can only be loosened, never tightened**. You can add new valid values but cannot remove previously valid ones. If incompatible changes are needed, they must be published under a new NSID. This ensures that older implementations can always validate data produced by newer implementations.

### Lexicons in Practice

When a client queries `app.bsky.actor.getProfile`, both sides (client and PDS) implement the same Lexicon schema. The PDS validates request parameters against the schema, executes the query, and returns a response conforming to the specified response schema. The client knows exactly what data to expect.

The separation between record schemas (stored data formats) and API schemas (query/mutation interfaces) enables important decoupling: multiple applications can define different APIs for reading and writing the same record types. Bluesky's App View defines `getFeedSkeleton`; a photo-focused App View could define entirely different endpoints while reading the same underlying record schemas.

---

## Data Repositories & Merkle Search Trees (MST)

### Repository Structure

Each user's data repository is a cryptographically signed collection of records organized into named collections. The structure has three layers:

1. **Records**: Individual JSON documents stored in CBOR binary encoding
2. **Merkle Search Tree**: Organizes records into a hierarchical tree with cryptographic hashes at each node
3. **Commit node**: Signs the root hash of the tree, providing a cryptographic commitment to the entire repository state

### How MSTs Work

Merkle Search Trees are the cryptographic backbone that makes AT Protocol's data self-authenticating. Key properties:

- **Ordered and insert-order-independent**: The tree structure is deterministic regardless of the order records were added
- **SHA-256 hashing** at each node with approximately **4-way fanout per layer**
- **Single root hash**: Any modification recomputes hashes up to the root, producing a new commit signature
- **Efficient synchronization**: Enables computation of minimal diffs between repository states
- **Verifiable by anyone**: Fetch the complete repository, validate all signatures, verify data integrity — no need to trust the PDS

The root hash, placed in the user's DID Document, serves as a globally accessible proof of the latest repository state. This is what makes the data **self-authenticating**: anyone can verify that records haven't been tampered with without trusting the hosting provider.

### Record Keys

Records within collections are identified by keys following specific schemes:

- **TIDs (Timestamp Identifiers)**: Base32-encoded values derived from creation time — the most common scheme, providing loose temporal ordering and efficient MST storage
- **NSIDs**: Reverse-DNS format for semantic meaning in keys
- **Literals**: Fixed well-known keys like `"self"` for singleton records (e.g., a user's profile)
- **Any**: Arbitrary strings for maximum flexibility

Keys are case-sensitive, ASCII-restricted, and limited to 512 characters.

### Binary Data (Blobs)

Media files are stored as separate blobs — unstructured binary data referenced by **CID (Content Identifier)**, a content-addressed hash. Records containing media include blob references with metadata (media type, size, CID). This separation allows App Views to fetch blobs from PDSes, process them (resize images, generate thumbnails), and serve them through CDNs independently of the repository's CBOR data structures.

---

## Streaming Architecture

### The Firehose

The firehose is the core protocol-level event stream: a continuous feed of **all public activity** in the network. It is produced by relays subscribing to PDSes and aggregating their repository update events into a single unified stream.

**Technical details:**
- **Encoding**: CBOR (Concise Binary Object Representation) — deterministic binary serialization suitable for cryptographic verification
- **Transport**: WebSocket connections
- **Wire format**: Headers indicate message types and sequence numbers; content is CBOR-encoded
- **Archive format**: CAR (Content Addressable aRchives) for repository snapshots
- **Throughput**: Hundreds to thousands of events per second as the network grows

**Who consumes the firehose:**
- Feed generators (indexing posts according to algorithmic criteria)
- Labelers (detecting content for moderation review)
- Search engines (crawling and indexing all content)
- Custom indexers (building specialized applications)

Every record in the firehose can be **cryptographically verified** by checking signatures against the creator's public keys published in their DID document. This is what makes the firehose trustworthy even when consumed through third-party relays.

### Jetstream

Jetstream is the developer-friendly alternative: a lightweight service (written in Go) that consumes the firehose and re-serves events as **JSON over WebSocket** instead of CBOR.

**Key features:**
- Simple JSON format instead of complex CBOR/CAR parsing
- Optional compression
- **Filtering by collection (NSID) or account (DID)** — subscribe only to posts and likes, ignoring profile updates and follows
- Multiple public instances operated by Bluesky; easy to self-host

**The trade-off:** Jetstream events do not include cryptographic signatures or Merkle tree nodes. The data is not self-authenticating — consumers must trust the Jetstream operator to provide unmodified data. For many use cases (feed generation, analytics, monitoring), this is acceptable. For applications requiring cryptographic verification, the raw firehose remains necessary.

---

## Aggregation Model

The aggregation model is perhaps AT Protocol's most distinctive architectural insight. It inverts the traditional database model:

**Traditional platform:** Application → Database (one system, one view)
**AT Protocol:** Multiple applications → Each maintains its own database → All sourced from the same firehose → Each database is a cache/materialized view of the source of truth

**The source of truth is always the user's PDS.** Every other database in the ecosystem — App Views, feed generators, search engines — is a derived view that can be rebuilt from scratch by replaying the firehose.

This model enables powerful features without centralization:
- **Search**: An independent service subscribes to the firehose, indexes content, and serves search queries
- **Custom feeds**: Feed generators maintain their own databases of posts matching algorithmic criteria
- **Notifications**: App Views compute notification state from the event stream
- **Analytics**: Third parties can build analytics services from public data

When a feed generator receives a request, it returns a **feed skeleton** — just a list of post URIs. The App View then **hydrates** these URIs into full post objects with author info, engagement metrics, and media. This separation means feed generators stay lightweight while App Views handle the heavy lifting of data assembly.

---

## The Ecosystem ("The Atmosphere")

The broader ecosystem of applications and tools building on AT Protocol is collectively called **The Atmosphere**. It demonstrates the protocol's core promise: multiple applications reading and writing the same underlying data.

### Applications

- **Bluesky**: The flagship microblogging platform — the application that drove AT Protocol's creation
- **Tangled**: Git hosting and code collaboration built on AT Protocol, enabling code repositories with AT Protocol identity
- **Leaflet**: Long-form publishing platform leveraging AT Protocol for identity and social features
- **Semble**: Event management built on AT Protocol identities
- **Wisp**: Experimental applications exploring the protocol's flexibility
- **Smoke Signal**: Event management and RSVP functionality using existing AT Protocol identities
- **WhiteWind**: Alternative blogging platform built on AT Protocol

### Cross-App Data Reuse

This is where the architecture's power becomes clear: **apps can piggyback on each other's data**. An event management app can use your existing social graph from Bluesky. A publishing platform can surface engagement from microblogging followers. Because all apps read from the same repositories and the same firehose, data created in one application context is available in another — no API integrations, no data export/import, no separate accounts.

### Developer Tools

- **pdsls**: Browse and inspect PDS contents
- **Taproot**: AT Protocol data explorer
- **atproto-browser**: Web-based repository browser
- **Graze**: Data visualization tool for AT Protocol
- **Quickslice**: Repository slicing and analysis
- **Tap**: Protocol debugging and inspection tool
- **SDKs**: TypeScript, Go, Python, Rust — handling CBOR encoding, firehose subscription, cryptographic verification, DID resolution

---

## Comparison with Solid Protocol

AT Protocol is not the first attempt at user-sovereign data storage. **Solid** (Social Linked Data), initiated by Tim Berners-Lee around 2009, shares remarkably similar goals but takes a different architectural path.

**Shared vision:** Both protocols aim for user-controlled data pods/servers where individuals own their data and applications request access to it. Both envision a world where switching applications doesn't mean losing data. Both separate data storage from application logic.

**Architectural differences:**
- Solid uses **Linked Data** (RDF, SPARQL) as its data model; AT Protocol uses **Lexicon-defined JSON/CBOR** records
- Solid uses **WebID** for identity; AT Protocol uses **DIDs**
- Solid emphasizes fine-grained **access control** (WAC/ACP); AT Protocol currently treats repository data as public
- Solid's data model is more general-purpose; AT Protocol is more opinionated and social-network-focused

**The convergence:** Both point toward the same **Open Social Web** — a future where data is user-controlled, applications are interchangeable, and identity is portable. AT Protocol has achieved faster adoption partly because it shipped a compelling application (Bluesky) first and designed the protocol around real-world scaling needs, while Solid has remained more academic and standards-focused.

---

## Limitations and Criticisms

### DID Inflexibility

The permanent nature of DIDs creates friction. Users cannot migrate between DID methods (`did:plc` to `did:web` or vice versa) without creating a new account and losing their entire social graph. The protocol tracks all relationships by DID. While future DID methods may be supported, the fundamental constraint — that a DID, once chosen, is permanent — remains a significant limitation for long-term account management.

### DNS Dependencies

Handles rely on DNS infrastructure — registrars, DNS providers, the DNS system itself. Domain seizures, DNS corruption, or regional DNS unavailability can break handle-based discovery. While the underlying DID remains accessible, the human-readable layer is fragile. This dependency on centralized naming infrastructure sits uncomfortably with the protocol's decentralization goals.

### Current Centralization Around Bluesky

The most consistent criticism: while the protocol *technically* permits anyone to operate relays and App Views, Bluesky operates the primary instances of both. Running production-grade relay infrastructure requires significant resources and expertise. Running an App View that processes the entire firehose at scale requires substantial computational investment. This creates *practical* centralization even where *technical* decentralization exists.

Direct messages in Bluesky are entirely centralized (routing through Bluesky's servers rather than operating peer-to-peer), and most users don't realize this distinction.

### Scaling Challenges for PDS Operators

While individual PDS operation is lightweight, the practical barriers to self-hosting remain substantial: domain management, DNS records, SSL certificates, server maintenance. Most users lack this expertise and remain dependent on third-party PDS providers. Critics argue the protocol didn't adequately solve the fundamental blocker to decentralized adoption — making self-hosting simple enough for non-technical users.

### Private Data and Encryption

All records in repositories are currently public. The protocol lacks robust mechanisms for private data sharing. While private data mechanisms are on the roadmap, adding encryption and access control will require significant architectural changes to protocol components that were designed around public data.

### Moderation Coordination

Despite the innovative labeler architecture, cross-application abuse is difficult to handle when different App Views implement different moderation policies. Users can create accounts, post problematic content through one App View, and have it replicate across others with different standards. Coordinated moderation across independent operators remains an unsolved problem.

---

## Key Takeaways for Mycelium

Each component of AT Protocol's architecture maps to a specific need in federated agent orchestration. Here's how the Mycelium project can leverage — and where it must extend — this infrastructure:

### Agent Identity via DIDs

Every agent in a Mycelium federation needs a persistent, verifiable identity that works across organizational boundaries. AT Protocol's DID system provides this directly. An agent with a `did:plc` identifier has:
- **Cryptographic proof of identity** that any peer can verify without a central authority
- **Portability** — the agent can migrate between hosting providers (PDSes) without losing its identity or reputation history
- **Signing keys** for authenticating all actions and data the agent produces

**Implication for Mycelium:** Agents should be first-class DID holders. An agent's DID document would declare its signing key, its hosting PDS endpoint, and potentially custom service endpoints advertising its capabilities.

### PDS as Agent Home

The PDS model maps directly to persistent agent state. An agent operating from its own PDS becomes a **data-sovereign entity** — not an ephemeral process spawned and discarded by an orchestrator, but a persistent participant with memory and history. Agent data stored on a PDS could include:
- **Capability declarations** (as Lexicon-defined records)
- **Work history** (completed tasks, inputs, outputs, time spent)
- **Reputation records** (stamps from other agents and orchestrators)
- **Current state** (in-progress tasks, dependencies, deadlines)

The low computational overhead of PDS operation means agents don't need expensive cloud infrastructure to maintain persistent presence. This enables an ecosystem where agents are as portable and independent as human users.

### Lexicons for Agent Capability Description

Lexicon schemas can formally describe agent capabilities in a machine-readable, discoverable format. Instead of natural language descriptions or hardcoded assumptions:

```
com.mycelium.agent.capabilities    → What work the agent can perform
com.mycelium.agent.requestWork     → Procedure to submit work to the agent
com.mycelium.agent.workResult      → Record type for completed work output
com.mycelium.reputation.stamp      → Record type for reputation attestations
```

The NSID governance model means any organization can define agent capability schemas under their own namespace without central coordination. Successful patterns can be standardized over time. Lexicon's backward compatibility rules (constraints only loosen) ensure that agents built against earlier schema versions continue to work as the ecosystem evolves.

### Firehose for Agent Discovery and Coordination

The firehose/Jetstream infrastructure provides the **ambient awareness** layer that federated agent orchestration requires. When agents complete work, update capabilities, or publish reputation stamps, these events flow through the firehose. Other agents and orchestrators can:
- Maintain real-time awareness of agent availability and specialization
- Discover new agents as they join the network
- Monitor work completion and reputation changes
- React to events without polling individual agent PDSes

This replaces centralized agent registries with organic, event-driven discovery — precisely the "wanted boards" model from the Wasteland architecture.

### App Views as Orchestration Layers

Mycelium orchestrators map naturally to the App View pattern. An orchestrator would:
- Subscribe to the firehose (or Jetstream, filtered by agent-related collections)
- Index agent capabilities, availability, and reputation into its own database
- Match work requests to qualified agents based on indexed data
- Serve coordination APIs to agents and human users

Multiple competing orchestrators could operate simultaneously, each indexing the same underlying data but implementing different matching algorithms, pricing models, or specialization strategies. Agents could participate in multiple orchestrators simultaneously through their single, portable identity.

### Labelers as Trust Infrastructure

The labeler model provides composable trust and reputation for agent networks. Independent labeler services could:
- Rate agent reliability based on work completion history
- Flag agents exhibiting problematic behavior (slow responses, incorrect outputs)
- Certify agent capabilities through verified testing
- Provide domain-specific trust signals (e.g., "trusted for production code" vs. "trusted for prototyping only")

Users and orchestrators subscribe to the trust labelers that align with their risk tolerance, creating a **market for trust** rather than a single centralized reputation authority.

### MSTs for Verifiable Agent Work Products

Merkle Search Trees ensure that every piece of agent-produced data is cryptographically verifiable. When an agent claims to have completed work, the evidence is:
- Signed with the agent's key
- Committed to the agent's repository
- Reducible to a single root hash that anyone can verify
- Tamper-evident — any modification breaks the hash chain

This provides the **verifiable evidence of work completion** that the Wasteland's reputation system requires. Orchestrators don't need to trust agents' self-reported results — they can cryptographically verify them.

### Gaps Mycelium Must Address

AT Protocol provides strong foundations but leaves several agent-specific challenges unaddressed:

1. **Private agent communication**: Agent-to-agent work coordination often involves private data (proprietary code, internal specifications). AT Protocol's current public-data-only model is insufficient. Mycelium needs encrypted channels — potentially leveraging DID-based key exchange.

2. **Real-time coordination**: The firehose provides near-real-time events, but agent orchestration may require sub-second coordination. WebSocket-based direct communication between agent PDSes, or a dedicated coordination protocol layer, may be necessary.

3. **Resource accounting**: AT Protocol has no native concept of computational costs, API quotas, or payment. Mycelium needs economic primitives for agent work — pricing, billing, resource limits.

4. **Capability verification**: Lexicons describe what agents *claim* to do. Mycelium needs mechanisms to *verify* those claims — sandboxed testing, benchmark suites, or reputation-gated capability assertions.

5. **Orchestrator authority**: The protocol treats all participants as peers. Agent orchestration requires hierarchy — task assignment, deadline enforcement, dependency management. These coordination semantics need to be built as application-layer Lexicons on top of the protocol's egalitarian foundation.

---

## Sources & References

- AT Protocol. "AT Protocol Documentation." *atproto.com*. https://atproto.com/
- AT Protocol. "AT Protocol Specifications." *atproto.com*. https://atproto.com/specs/atp
- AT Protocol. "Lexicon: Schema-Driven Interoperability." *atproto.com*. https://atproto.com/specs/lexicon
- AT Protocol. "Personal Data Server (PDS) Self-Hosting Guide." *atproto.com*. https://atproto.com/guides/self-hosting
- AT Protocol. "Decentralized Identifiers (DID)." *atproto.com*. https://atproto.com/specs/did
- AT Protocol. "Data Repositories." *atproto.com*. https://atproto.com/specs/repository
- AT Protocol. "Event Streams (Firehose)." *atproto.com*. https://atproto.com/specs/event-stream
- Bluesky Social. "AT Protocol — GitHub Repository." *github.com*. https://github.com/bluesky-social/atproto
- Abramov, Dan. "Open Social." *overreacted.io*, 2025. https://overreacted.io/open-social/
- Abramov, Dan. "A Social Filesystem." *overreacted.io*, 2025. https://overreacted.io/a-social-filesystem/

*Next in series: Report 3 — Synthesis: Bridging Protocols and the Architecture of Federated Agent Networks*
