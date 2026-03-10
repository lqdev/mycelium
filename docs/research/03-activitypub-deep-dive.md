# ActivityPub Deep Dive: Federation, Identity, and the Fediverse at Scale

**Mycelium Research Report #3**
*Part of the Mycelium series — investigating federated agent orchestration on decentralized social protocols*

---

## Overview & Philosophy

ActivityPub is a W3C-recommended specification (W3C Recommendation, January 2018) for decentralized social networking. It defines how independent servers can maintain user accounts and exchange social data through standardized protocols, forming the technical backbone of what has come to be called the **fediverse** — a constellation of interoperating social platforms spanning microblogging, video sharing, photo sharing, event coordination, link aggregation, and more.

The conceptual model is deliberately analogous to email. Just as anyone can run an email server and exchange messages with any other email server through SMTP, anyone can run an ActivityPub server and exchange social activities with any other conforming server. Users create accounts on a server of their choosing (or one they operate themselves), and their server handles the business of sending and receiving social data on their behalf. There is no central authority, no single company controlling the namespace, and no platform lock-in — at least in theory.

This email-like philosophy carries a specific set of trade-offs that distinguish ActivityPub from both centralized platforms and from alternative decentralized approaches like the AT Protocol. ActivityPub prioritizes **server autonomy** and **direct peer-to-peer federation** between any two compatible implementations. Each server is a sovereign node that decides its own moderation policies, feature set, and federation relationships. The protocol provides the shared language; the servers provide the governance.

For the Mycelium project, ActivityPub represents the most battle-tested approach to decentralized social infrastructure. With millions of users, hundreds of implementations, and billions of federated objects exchanged, it is the largest working proof that federation at social-network scale is achievable. Understanding its architecture, strengths, and hard-won lessons is essential groundwork for designing federated agent orchestration systems.

---

## Architecture

### The Inbox/Outbox Model

ActivityPub's most distinctive architectural contribution is its **inbox/outbox model** for distributing social activities across federated servers.

Every actor in the system exposes two critical endpoints:

- **Inbox**: Receives incoming activities from other servers and actors. When an actor on Server A wants to interact with an actor on Server B — follow them, reply to their post, like their content — Server A constructs an Activity object and sends it via HTTP POST to the target actor's inbox URL. The receiving server validates the request, processes the activity, and updates its local state accordingly.

- **Outbox**: Records activities that the actor creates and publishes. When a user writes a post or performs any action, their server records it in their outbox and uses it as the source for notifying followers and relevant actors across the federation. The outbox is an ordered collection that serves as both a historical record and a feed that remote servers can query.

This push-pull architecture creates a resilient model where the outbox provides the **source of truth** for an actor's activities while inboxes serve as **distributed notification endpoints**. Federation happens asynchronously — there is no requirement for real-time synchronization between all servers, and temporary unavailability of a remote server doesn't invalidate the sending server's record.

**Server-to-Server (S2S) federation** is the primary mode of ActivityPub operation. When Server A delivers an activity to Server B's inbox, Server B validates the HTTP Signature, verifies the sending actor exists, checks addressing fields, and only then processes the activity. This is the mechanism that knits the fediverse together — every follow, every reply, every like crossing server boundaries flows through this inbox delivery pipeline.

### Actors

Actors are the primary unit of identity and agency in ActivityPub. An actor represents any entity capable of performing activities — a person, an organization, a bot, an application, or any other autonomous agent. Each actor is identified by a globally unique HTTP URI, typically following the format of their home server address plus a username path (e.g., `https://mastodon.social/users/alice`).

An actor's profile contains metadata including name, description, avatar, and critically, the URLs for their inbox and outbox endpoints. The actor URI serves double duty: it is both the **identity** of the actor and the **authority** that determines where that actor's content originates. This coupling of identity to server infrastructure is one of ActivityPub's most consequential design decisions, with implications explored in the Identity and Portability sections below.

Actor types defined by ActivityStreams include **Person**, **Organization**, **Service**, **Application**, and **Group** — each inheriting the same inbox/outbox mechanics but representing different categories of network participants.

### Activities

Activities are the fundamental unit of meaning in ActivityPub. An activity is a structured JSON object describing an action performed by one actor toward another actor or object. The ActivityStreams 2.0 vocabulary defines a rich set of activity types:

- **Create**: Publishing new content (a Create activity wrapping a Note, Article, etc.)
- **Update**: Modifying existing content
- **Delete**: Removing content
- **Follow**: Establishing a subscription relationship
- **Like**: Expressing approval of an object
- **Announce**: Sharing/boosting another actor's content (analogous to retweet/reblog)
- **Accept/Reject**: Responding to requests (e.g., accepting a Follow request)
- **Add/Remove**: Managing collections
- **Undo**: Reversing a previous activity (unfollowing, unliking)

Each activity carries properties for the `actor` performing the action, the `object` being acted upon, an optional `target`, and addressing fields (`to`, `cc`, `bcc`, `bto`, `audience`) that determine who receives notification. This activity-centric model allows the protocol to represent social interactions from simple likes to complex workflows without requiring protocol changes for each new interaction type.

### Objects

Objects represent the entities acted upon within the protocol — the content that populates social feeds:

- **Note**: Short textual posts (the primary currency of microblogging platforms like Mastodon)
- **Article**: Longer-form written content
- **Image**, **Video**, **Audio**: Media objects
- **Event**: Calendar events (used by platforms like Mobilizon)
- **Page**: Web page references (used by link aggregation platforms like Lemmy)
- **Collection**, **OrderedCollection**: Groups of objects

Each object has a unique identifier (a URL on its origin server), a type, and properties appropriate to that type. Objects also carry their own addressing information, creating a layered system where both the activity and the object it wraps can specify audience targeting. This separation enables fine-grained control over content distribution but introduces complexity — a tension that runs through much of ActivityPub's design.

### Client-to-Server (C2S)

The ActivityPub specification defines a **Client-to-Server (C2S)** protocol layer that allows client applications to interact with a user's server through standardized API calls. In theory, C2S would enable any compatible client to work with any compatible server, similar to how any IMAP email client can connect to any IMAP server.

In practice, C2S has seen minimal adoption. Most implementations use custom, platform-specific APIs for client-server communication (Mastodon has its own REST API, Pleroma has its own, etc.). The C2S specification proved too generic for the specific needs of diverse platforms, and the practical benefits of a standardized client API were outweighed by the flexibility of custom APIs tailored to each platform's feature set. This represents a notable gap between ActivityPub's architectural vision and its real-world implementation.

---

## Federation Model

### Server-to-Server Communication

The federation mechanism is ActivityPub's most complex and operationally critical component. The fundamental operation is straightforward: one server discovers the inbox address of a remote actor and delivers activities via HTTP POST. The details, however, are where complexity accumulates.

When a local actor performs an action requiring federation — posting a public message, following a remote user, replying to remote content — the sending server must:

1. Identify all recipients based on the activity's addressing properties
2. Retrieve actor profiles for those recipients (via HTTP GET to their actor URIs)
3. Extract inbox endpoints from those profiles
4. Deliver the activity to each inbox via authenticated HTTP POST

The request body contains the activity as JSON-LD, with the Content-Type header set to `application/ld+json; profile="https://www.w3.org/ns/activitystreams"`.

### HTTP Signatures for Authentication

Authentication between servers is primarily handled through **HTTP Signatures**, which provide cryptographic proof that the sending server holds the private key associated with the claimed actor. The signature is computed over specific headers — `Date`, `Host`, `Digest`, `Content-Type`, and the request target — creating a cryptographic binding between the request and its contents.

The receiving server validates the signature by fetching the sender's actor profile to retrieve their public key, then verifying the signature against the request headers. This provides reasonable assurance that the activity genuinely originated from the claimed server, though it authenticates only the **transport layer** rather than the persistent object itself — a limitation that FEP-8b32 (Object Integrity Proofs) aims to address.

### Fanout Delivery

One of ActivityPub's most significant scaling challenges is **fanout delivery**. When a user publishes a public post, their server must deliver that activity to every follower's server individually. A popular account with followers distributed across thousands of servers requires thousands of individual HTTP POST deliveries for every single post.

This computational burden scales multiplicatively with network growth. The sending server bears the full cost of delivery — there is no shared infrastructure, no broadcast mechanism, no content delivery network built into the protocol. Servers must implement retry logic for failed deliveries (the spec recommends retries but doesn't mandate specific strategies), and different implementations handle failures differently — some retry for weeks, others give up after minutes.

The fanout problem represents a fundamental architectural tension: the same design that ensures no central authority controls content distribution also means that every server must individually bear the cost of reaching its users' audiences. This stands in stark contrast to AT Protocol's firehose/relay model, where a central aggregation layer handles the distribution problem at the cost of introducing a potential centralization point.

### Content-Addressed vs. Origin-Addressed Objects

ActivityPub objects are **origin-addressed** — identified by URLs on the server that created them. A post's identity is inseparable from the server that hosts it (e.g., `https://mastodon.social/users/alice/statuses/12345`). This means the object's existence and accessibility depend on the continued operation of that specific server.

This contrasts with **content-addressed** systems where objects are identified by hashes of their content, making them retrievable from any source that holds a copy. FEP-ef61 (Portable Objects) attempts to move ActivityPub toward content-addressable objects, but this represents a fundamental architectural shift that challenges the protocol's HTTP-centric design.

---

## JSON-LD and Activity Streams 2.0

### Linked Data Foundations

ActivityPub's interoperability layer is built on **JSON-LD** (JSON for Linked Data) and the **Activity Streams 2.0** vocabulary. JSON-LD embeds semantic meaning into JSON documents through the `@context` mechanism, which maps property names to globally unique namespace URIs. A standard ActivityPub object begins with:

```json
{
  "@context": "https://www.w3.org/ns/activitystreams",
  "type": "Note",
  "content": "Hello, fediverse!",
  "attributedTo": "https://example.com/users/alice"
}
```

The `@context` declaration tells conforming processors that `type`, `content`, and `attributedTo` should be interpreted according to the Activity Streams vocabulary, with specific semantic meanings defined by the W3C specification.

### Extensibility in Theory

JSON-LD's extensibility mechanism allows implementers to define custom properties by adding their own namespace contexts. A platform wanting to add emoji reactions can define a custom `@context` entry mapping a property name to their namespace URI, enabling processors that understand that namespace to correctly interpret the extension while processors that don't can safely ignore it.

This theoretical extensibility is powerful — any implementer can innovate without requiring protocol-level consensus. Misskey defines custom properties for emoji reactions and quote posts under its own namespace. Mastodon extends the vocabulary with properties for featured collections and focal points on images. Any platform can do the same.

### The Practical Gap

In practice, many implementers **ignore the JSON-LD aspects entirely**, treating ActivityPub payloads as flat JSON with some conventions about property names. This pragmatic approach works for most social media use cases — you don't need a full JSON-LD processor to handle a Note with `content` and `attributedTo` fields. But it creates a growing gap between the protocol's theoretical capabilities and its practical implementation.

The JSON-LD debate has become a persistent source of tension in the community. Advocates argue that linked data principles enable powerful semantic reasoning and future extensibility. Critics counter that JSON-LD adds significant complexity for diminishing returns — most social networking interactions don't benefit from formal semantic reasoning, and the cognitive overhead of understanding JSON-LD discourages new implementers.

The result is a protocol where the formal specification says JSON-LD, but the practical ecosystem largely treats the data as conventional JSON with well-known property names. This works until it doesn't — edge cases around extension namespaces, property compaction, and context resolution can produce subtle interoperability bugs that are difficult to diagnose.

---

## Current Ecosystem

The breadth of ActivityPub's ecosystem demonstrates the protocol's flexibility, while also illustrating the interoperability challenges that come with diverse implementations.

### Mastodon

The **flagship** of the fediverse. Launched in 2016, Mastodon pioneered federated microblogging with a familiar, Twitter-like interface. It established ActivityPub as viable for real-world social networking and remains the gravitational center of the ecosystem, with millions of users across hundreds of independent instances. Mastodon's implementation choices have become de facto standards that other platforms must accommodate, sometimes creating tension between "what the spec says" and "what Mastodon does."

### Pleroma / Akkoma

Alternative microblogging platforms built in Elixir, prioritizing lightweight deployment and customization. Pleroma targets tech-savvy users wanting greater control; Akkoma (a Pleroma fork) offers faster development cycles and features like custom emoji reactions and Misskey-flavored markdown. These platforms demonstrate that ActivityPub enables multiple competing implementations to coexist and interoperate despite different languages, architectures, and design philosophies.

### PeerTube

Federated **video sharing**, extending ActivityPub beyond text into media-heavy use cases. PeerTube federates video metadata between instances and maintains compatibility with text-based platforms — a PeerTube video can appear in a Mastodon timeline, and Mastodon users can comment on it. This cross-platform federation is one of ActivityPub's most compelling demonstrations.

### Pixelfed

Federated **photo sharing** with an Instagram-like experience. The release of native iOS and Android applications demonstrated that ActivityPub can support consumer-grade mobile experiences comparable to centralized platforms — addressing a long-standing concern about whether federated protocols can serve mainstream users.

### Lemmy

Federated **link aggregation and discussion**, creating a Reddit-like experience. Lemmy emphasizes community organization over individual feeds, with content grouped into communities that users on any instance can participate in. This demonstrates ActivityPub's capacity to support social structures beyond the microblogging paradigm.

### Mobilizon

Federated **event coordination and group management**. Mobilizon implements only the S2S component of ActivityPub, focusing on federation between Mobilizon instances while supporting follow relationships with Mastodon accounts. This selective implementation illustrates how platforms can adopt the portions of the spec relevant to their domain.

### Meta's Threads

The corporate elephant in the room. Launched in 2023 as a Twitter alternative, Threads began optional ActivityPub federation in 2024 — a watershed moment for the protocol's mainstream legitimacy. Users 18+ with public profiles can choose to federate, allowing cross-platform following and sharing. The integration exposed numerous interoperability gaps (particularly around quote posts) and created community tension around whether a corporate giant's participation strengthens or threatens the fediverse.

### Friendica

A **multi-protocol bridge** supporting ActivityPub, Diaspora, and RSS. Friendica occupies a unique niche as a connector between different social networking ecosystems, allowing users to maintain presence across multiple networks through a single instance.

### Hubzilla

Pioneer of **nomadic identity** through the Zot protocol. Hubzilla supports multiple protocols and uniquely emphasizes identity portability and sophisticated access control. Its nomadic identity architecture — where a user's identity can be cloned across multiple servers — represents one of the most ambitious attempts to solve the identity portability problem in federated systems.

---

## Fediverse Enhancement Proposals (FEPs)

### Community-Driven Evolution

The core ActivityPub specification evolves slowly through formal W3C processes, but the ecosystem's feature needs outpace that cadence. **Fediverse Enhancement Proposals (FEPs)** provide a community-driven mechanism for proposing extensions, modifications, and clarifications. Any community member can submit a FEP, discuss it publicly, and seek consensus — no permission from a central authority required.

As of early 2026, hundreds of FEPs exist in various states of proposal, discussion, and implementation. Key proposals include:

### Notable FEPs

- **FEP-7628 (Move Actor)**: Proposes mechanisms for actors to migrate between servers while maintaining their social graph. The migrating actor posts a `Move` activity from the old account and creates a forwarding mechanism. Limitations: only works if the old account is still accessible, and there's no guarantee all servers will honor the redirect.

- **FEP-ef61 (Portable Objects)**: Proposes making objects identifiable independent of their hosting server — a fundamental shift from origin-addressed to content-addressed objects. Aims to solve the problem of content becoming inaccessible when servers shut down. Architecturally ambitious but controversial and difficult to implement with backward compatibility.

- **FEP-8b32 (Object Integrity Proofs)**: Proposes cryptographic signatures on ActivityPub objects themselves (not just the HTTP transport), allowing verification of an object's authenticity independent of how it was transmitted. Addresses a fundamental limitation of HTTP Signatures, which only authenticate the delivery, not the persistent content.

- **FEP-c0e0 (Emoji Reactions)**: Standardizes emoji reactions that emerged from Misskey and spread across platforms. Illustrates how platform-specific innovations get formalized through the FEP process.

- **FEP-9967 (Polls)**: Standardizes structured poll/question functionality across platforms.

### Strengths and Weaknesses of the FEP Process

The FEP process's **strength** is its openness — anyone can propose, and innovation isn't bottlenecked by a standards body. Its **weakness** is fragmentation. With hundreds of FEPs and different implementers supporting different subsets, the practical ecosystem becomes increasingly divergent. Servers implementing incompatible FEP combinations may fail to properly exchange certain content types, and there's no mechanism to enforce adoption.

### W3C Social Web Working Group

Recognizing the fragmentation risk, the W3C established a new **Social Web Working Group** in early 2026 with an explicit mandate to release backward-compatible iterations of ActivityPub and Activity Streams specifications. The working group aims to incorporate lessons from years of practical implementation, standardize proven extensions, and clarify undefined behaviors — while coordinating (rather than replacing) the community-driven FEP process. The charter targets Q3 2026 for initial specification updates.

---

## The Nomadic Identity Problem

### Identity Tied to Servers

ActivityPub's HTTP-based architecture ties identity directly to domain names and server infrastructure. An actor's unique identifier is a URL controlled by their home server: `https://mastodon.social/@alice` means Alice's identity is inseparable from the `mastodon.social` domain. If that server shuts down, Alice's identity — and everything associated with it — becomes inaccessible. Followers lose the ability to follow her even if she reappears on another server. Her content URLs become dead links. Her social graph fragments.

This is the **nomadic identity problem**: in a federated system where servers are operated by volunteers, small organizations, or companies that may not last forever, tying identity to server URLs creates an existential fragility. Server death equals identity death.

### Hubzilla and the Zot Protocol

The **Hubzilla** project and its **Zot protocol** pioneered the most ambitious solution to nomadic identity within the ActivityPub ecosystem. In Hubzilla's model, a user's primary identity exists independently of any particular server. That identity can be **cloned** across multiple servers — when the primary hub becomes unavailable, the user can log into any clone and continue their social activities without losing identity or social graph.

This architecture requires careful **cryptographic key management**: the user's identity is verifiable through possession of a private key that can sign activities and prove ownership from any instance claiming to represent that identity. The private key, not the server URL, becomes the root of identity.

While technically impressive, Hubzilla's approach hasn't achieved broad adoption. The cognitive overhead of managing cloned identities across multiple servers is significant, and the implementation complexity has limited Hubzilla to a relatively small, technically sophisticated user base.

### AT Protocol's DIDs: A More Elegant Solution?

AT Protocol takes a different approach with **Decentralized Identifiers (DIDs)**. A user's identity is a DID that is independent of any particular server. Users can change their Personal Data Server (PDS) — the server hosting their data — while keeping the same DID, maintaining their identity and social graph across migrations.

This is arguably more elegant than Hubzilla's clone-based approach: instead of maintaining multiple synchronized copies of identity, AT Protocol separates the identity layer (DIDs) from the hosting layer (PDSes) entirely. However, it introduces its own complexity around DID resolution and key management, and the practical implementation still depends heavily on Bluesky's infrastructure.

### FEP-7628's Limitations

ActivityPub's native answer to migration — **FEP-7628** — proposes a `Move` activity that redirects followers from an old account to a new one. The limitations are significant:

- Only works if the old account is still accessible (useless if the server died unexpectedly)
- No guarantee that all remote servers will honor the redirect
- Doesn't address content migration — previous posts remain on old URLs
- Doesn't preserve social context (reactions, replies from others)

FEP-7628 is migration as a best-effort courtesy, not a robust portability mechanism.

---

## Data Portability Challenges

### The Content Migration Problem

Even when identity can be migrated, **content portability** presents distinct challenges. In ActivityPub, every post, every image, every interaction is identified by a URL on the origin server. Moving to a new server means new URLs, which breaks:

- Existing links and references from other users' posts
- Conversation threads (replies point to original URLs via `inReplyTo`)
- Boosts/announces that reference the original URL
- Search engine indexes and external references

### Lost Social Context

The most painful limitation is the loss of **social context** during migration. Comments and reactions from remote users are anchored to the old URLs and don't automatically transfer. Moving servers means losing the accumulated engagement on every post — the likes, the replies, the conversation threads. From the user's perspective, migration is a partial social death even when the content itself is preserved.

### FEP-ef61 and Architectural Tension

FEP-ef61 (Portable Objects) attempts to address this by making objects identifiable through content hashing rather than server URLs. This would allow an object to be recognized as "the same post" regardless of where it's hosted. But this represents a **fundamental architectural change** — moving from an origin-addressed to a content-addressed model — that challenges ActivityPub's HTTP-centric design and has proven difficult to implement while maintaining backward compatibility.

### The Fundamental Challenge

The data portability problem in ActivityPub is not a bug to be fixed but a **consequence of architectural choices**. The protocol is server-centric by design — servers are the units of authority, identity, and content hosting. Making data truly portable requires either weakening the server's role (which changes the protocol's fundamental character) or building elaborate migration machinery on top of a foundation not designed for it. This tension between server-centric architecture and user-centric data ownership remains unresolved.

---

## Comparison with AT Protocol

Understanding ActivityPub's design choices is sharpened by comparison with AT Protocol, which makes fundamentally different trade-offs.

### Email Model vs. Web Model

ActivityPub uses an **email model**: independent servers maintain accounts and send messages to each other. Each server is a peer that can directly communicate with any other.

AT Protocol uses a **web model**: accounts are independent data repositories on Personal Data Servers, and applications (AppViews) aggregate data from those repositories to create user experiences. The network depends on indexing infrastructure to function — users can't discover content through their PDS alone.

### Identity Architecture

| Dimension | ActivityPub | AT Protocol |
|-----------|-------------|-------------|
| Identity root | Server URL | DID (Decentralized Identifier) |
| Migration | Best-effort redirect (FEP-7628) | Change PDS, keep DID |
| Server death impact | Identity lost | Identity survives |
| Complexity | Simple (URL = identity) | Complex (DID resolution, key management) |

### Content Distribution

| Dimension | ActivityPub | AT Protocol |
|-----------|-------------|-------------|
| Delivery model | Fanout (push to each follower's server) | Firehose/relay (aggregators pull from all PDSes) |
| Scaling burden | On the sending server | On relay/AppView infrastructure |
| Centralization risk | Distributed (no single point) | AppView/relay concentration |
| Delivery guarantee | Best-effort with retries | Aggregator-dependent |

### Extensibility

| Dimension | ActivityPub | AT Protocol |
|-----------|-------------|-------------|
| Extension mechanism | JSON-LD namespaces | Lexicon schemas |
| Validation | Loose (many ignore JSON-LD) | Strict (schema-validated) |
| Innovation speed | Fast (anyone can extend) | Controlled (Lexicon registry) |
| Fragmentation risk | High | Lower |

### The Verdict

Neither protocol has won. ActivityPub has the **maturity advantage** — millions of users, W3C standardization, a diverse ecosystem proving the protocol's flexibility. AT Protocol has the **architectural advantage** — portable identity, cleaner separation of concerns, and a data model designed from the start for the problems ActivityPub is now retrofitting solutions for.

They serve different trade-offs. ActivityPub optimizes for **server autonomy and direct federation**. AT Protocol optimizes for **user data portability and application-layer innovation**. Both have centralization risks — ActivityPub around dominant instances (mastodon.social), AT Protocol around dominant AppViews (Bluesky's infrastructure).

---

## Strengths

### Maturity and Standardization

ActivityPub is a **W3C Recommendation** — the highest level of endorsement from the web's standards body. It has formal specifications, established governance, and years of real-world deployment. This maturity provides confidence that the protocol is battle-tested and that its failure modes are understood, even if not all are resolved.

### True Peer-to-Peer Federation

Any two compatible ActivityPub servers can directly exchange content without intermediaries. There is no required relay, no mandatory indexer, no infrastructure dependency beyond basic HTTP. This is genuine peer-to-peer federation — each server can communicate with any other, limited only by network connectivity and protocol compliance.

### Ecosystem Diversity

The diversity of platforms built on ActivityPub — microblogging, video, photos, events, link aggregation, groups — proves the protocol's flexibility. ActivityPub isn't locked to one use case. The same protocol underpins Mastodon's Twitter-like experience, PeerTube's YouTube-like video sharing, Pixelfed's Instagram-like photos, and Lemmy's Reddit-like communities. This breadth is unmatched by any competing decentralized protocol.

### Proven Scale

Millions of users. Hundreds of implementations. Billions of federated objects. ActivityPub has proven that decentralized social networking works at meaningful scale. This isn't a theoretical exercise — it's a functioning global network that people use daily as their primary social media platform.

---

## Weaknesses

### Scalability Challenges

Fanout delivery is ActivityPub's Achilles' heel at scale. A popular account posting to millions of followers distributed across thousands of servers requires thousands of individual HTTP POST deliveries **per post**. The sending server bears the full computational cost. There's no broadcast primitive, no shared delivery infrastructure, no CDN for activities. Servers must implement their own retry strategies, queue management, and failure handling — and the spec doesn't mandate how.

### Identity Fragility

Identity tied to server infrastructure means identity dies with the server. Migration mechanisms (FEP-7628) are best-effort, only work when the old server is still accessible, and don't preserve content or social context. For a protocol designed to free users from platform lock-in, the practical reality is that users are locked into their server's continued existence.

### JSON-LD Complexity

JSON-LD adds significant conceptual and implementation overhead for benefits that most social networking use cases don't realize. The gap between the protocol's formal linked-data specification and the flat-JSON reality of most implementations creates a two-tier ecosystem where some implementations properly handle JSON-LD semantics and others don't — leading to subtle interoperability bugs.

### Ecosystem Fragmentation

The FEP process enables innovation but also fragmentation. Different platforms support different subsets of FEPs, different extension namespaces, and different interpretations of underspecified behavior. Quote posts exist in at least three incompatible formats. Emoji reactions work differently across platforms. Post editing propagates inconsistently. The result is a federation that works well for basic interactions (posts, follows, likes) but breaks down for more sophisticated features.

### Privacy and Security Gaps

Private messages traverse between servers as plaintext, visible to any server administrator with database access. There is no native encryption support. End-to-end encryption proposals exist but haven't achieved broad implementation due to the complexity of maintaining encryption across distributed infrastructure and the key management challenges of a system where identity is tied to server URLs.

Security vulnerabilities — including critical impersonation bugs where remote actors could forge identities through improperly validated ActivityPub objects — have highlighted the importance of rigorous validation, but the decentralized nature of the protocol means security patches must be adopted voluntarily by each server operator.

---

## Key Takeaways for Mycelium

ActivityPub's decade of real-world operation provides invaluable lessons for the Mycelium project's vision of federated agent orchestration. These are the lessons we extract:

### 1. Federation Works — But Fanout Doesn't Scale

ActivityPub proves that federation at social-network scale is achievable. Millions of users exchange billions of objects across hundreds of independent servers. But the fanout delivery model — where the sending server bears the full cost of reaching every recipient — becomes a scaling bottleneck for popular actors. **For Mycelium**: Agent orchestration will need a distribution model that doesn't require the orchestrating agent to individually contact every participant. AT Protocol's relay/firehose approach or a pub-sub model may be more appropriate for agent task distribution than ActivityPub's direct inbox delivery.

### 2. Identity Must Be Independent of Infrastructure

ActivityPub's hardest unsolved problem is identity portability. Tying identity to server URLs creates existential fragility — server death means identity death. Years of FEP proposals and nomadic identity experiments haven't fully solved this within ActivityPub's architecture. **For Mycelium**: Agent identity must be built on a foundation independent of any particular server or hosting provider. DID-based identity (as AT Protocol uses) is the stronger foundation. An agent's identity and reputation should survive the shutdown of any individual infrastructure component.

### 3. Extensibility Needs Guardrails

ActivityPub's JSON-LD extensibility is theoretically unlimited but practically fragmenting. Any implementer can add any extension, leading to a proliferation of incompatible approaches to the same features. **For Mycelium**: Agent capability advertisement and task schema definitions need a structured extensibility mechanism (closer to AT Protocol's Lexicon schemas than ActivityPub's free-form JSON-LD extensions) that enables innovation while maintaining interoperability. A capability registry with versioned schemas would prevent the fragmentation that plagues ActivityPub's extension ecosystem.

### 4. The Spec Must Define Behavior, Not Just Format

Much of ActivityPub's interoperability pain comes from **underspecified behavior** — the spec defines the data format but leaves processing semantics ambiguous. Different implementations make different choices about edge cases, creating a federation that works for common paths but breaks for anything unusual. **For Mycelium**: The agent orchestration protocol must specify not just message formats but expected processing behavior. What happens when a task assignment is received but the agent is at capacity? What happens when a result is delivered to an agent that didn't request it? These behavioral semantics must be defined, not left to implementers.

### 5. Governance is a Feature, Not an Afterthought

ActivityPub's community-driven FEP process enables rapid innovation but lacks enforcement mechanisms. The result is an ecosystem where interoperability is aspirational rather than guaranteed. The 2026 W3C Social Web Working Group formation — eight years after the initial recommendation — represents a belated recognition that formal governance is necessary. **For Mycelium**: The governance model for agent orchestration standards should be designed from the start, not retrofitted. A clear process for proposing, validating, and adopting protocol extensions will prevent the fragmentation that ActivityPub now struggles to remediate.

### 6. Server Autonomy and Moderation Are Inseparable

ActivityPub's greatest social achievement is enabling **server-level governance** — each instance sets its own moderation policies, content rules, and federation relationships. This maps well to agent orchestration, where different organizations will have different policies about which agents can participate, what data can be shared, and what tasks are permitted. **For Mycelium**: The orchestration protocol should support organization-level policy enforcement and selective federation, not assume a flat global namespace of cooperating agents.

### 7. Multi-Protocol Bridges Are Inevitable

Friendica's multi-protocol approach and the emerging Bridgy Fed project demonstrate that protocol boundaries don't have to be walls. The fediverse's experience shows that bridge infrastructure becomes essential as ecosystems grow. **For Mycelium**: Rather than betting exclusively on one protocol (AT Protocol or ActivityPub), the agent orchestration layer should be designed with protocol bridging as a first-class concern. Agents built on AT Protocol should be able to coordinate with agents on ActivityPub-based systems through well-defined bridge interfaces.

### 8. Start With the Hard Problems

ActivityPub started with the easy problems (posting and following) and deferred the hard ones (identity portability, data migration, end-to-end encryption, content integrity). Years later, those deferred problems have become architectural debt that's enormously expensive to pay down within the existing design. **For Mycelium**: Agent identity portability, task provenance, result integrity, and capability verification should be core protocol features from day one, not future enhancements. The cost of retrofitting these concerns onto an established protocol — as ActivityPub is now learning — is far higher than building them in from the start.

---

## Sources & References

- W3C. "ActivityPub — W3C Recommendation." *w3.org*, January 2018. https://www.w3.org/TR/activitypub/
- W3C. "Activity Streams 2.0." *w3.org*. https://www.w3.org/TR/activitystreams-core/
- "ActivityPub Rocks — Resources and Community." *activitypub.rocks*. https://activitypub.rocks/
- AT Protocol. "AT Protocol Documentation." *atproto.com*. https://atproto.com/
