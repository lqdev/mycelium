# Report 04: The Social Filesystem — How AT Protocol Reimagines Data Ownership

## A Deep Dive Synthesis of Dan Abramov's "Open Social" and "A Social Filesystem"

*Mycelium Research Series — Report 4*

---

## Executive Summary

This report synthesizes the two landmark articles by Dan Abramov — "Open Social" (overreacted.io/open-social/) and "A Social Filesystem" (overreacted.io/a-social-filesystem/) — into a unified analysis of the conceptual foundations underlying the AT Protocol. Together, these articles articulate the most compelling vision yet for why decentralized social infrastructure matters, how it works in practice, and what it makes possible. The central thesis is deceptively simple: **what if social data behaved like files?** The implications of this question, rigorously explored through Abramov's filesystem metaphor, reveal an architecture where users own their data, apps become reactive views over that data, and the boundaries between applications dissolve into a shared, interconnected web of typed JSON records. For the Mycelium project, this conceptual framework is foundational — it suggests that agents, like users, can maintain their own "everything folders" of work artifacts, capability declarations, and coordination records, all governed by shared schemas and discoverable across the network.

---

## The Core Analogy: Files → Social Records

### The Personal Computing Paradigm

Abramov opens "A Social Filesystem" with a deceptively simple observation: remember files? In the paradigm of personal computing, you write a document, hit save, and the file is on your computer. It's yours. You can inspect it, send it to a friend, open it with other apps. Files belong to the user, not to the application that created them.

This is so fundamental that we barely notice it, but the implications are profound. When you create an SVG in Excalidraw, that file can be displayed in any browser, edited in Illustrator, transformed by a command-line tool, or embedded in a webpage. The file format is the contract — the API, if you will — that enables interoperability between entirely unrelated applications. No integration was built. No partnership was signed. The applications simply agree on what an SVG looks like.

The key principle Abramov articulates: **"What we make with a tool does not belong to the tool."** Your documents, your images, your spreadsheets — these are your artifacts. Apps may come and go, but files stay. The application is a lens through which you create and view your data, not a vault that imprisons it.

### From Files to Social Data

So what happened to this principle when computing went social? When Alice posts on Instagram or Bob tweets on X, they are creating data — but that data doesn't behave like a file. It can't be inspected independently, can't be opened with another app, can't be migrated when the platform turns hostile. The data is trapped as rows in somebody else's database.

Abramov's provocation: **what if social data behaved like files?** What if a "Tumblr post" or an "Instagram follow" were file formats — structured data that lived in your folder, created by apps on your behalf but ultimately belonging to you?

This isn't merely an analogy. It's a design specification, and it's exactly what AT Protocol implements in production.

---

## The "Everything Folder"

### A New Mental Model

Imagine each user has an "everything folder" containing all things ever POSTed by their online persona. Every post you've written, every like you've given, every follow you've initiated, every recipe you've shared — all of it lives in your folder as discrete, inspectable records.

In this model, the relationship between apps and data inverts:

- **Today**: Apps own databases. Your data is scattered across corporate servers. To access it, you go through the app.
- **Everything folder**: You own your data. Apps read and write to your folder on your behalf. Apps are views over your data, not containers for it.

This is a subtle but revolutionary shift. When files are the source of truth, **apps become reactive to files**. An app's database isn't the canonical store of your data — it's a derived, cached materialized view. If you delete a record from your folder, it disappears from every app that was displaying it. If you create a record, every app subscribed to that record type can react to it.

Abramov demonstrates this viscerally in his article: he creates a Bluesky post by writing a raw JSON record directly into his repository using the pdsls developer tool. No Bluesky client was involved. Yet the post appears in Bluesky's timeline instantly — because Bluesky is simply a reactive view over the records in users' repositories. Files are the source of truth; the app "reacts" to them.

### This Is Not Theoretical

The "everything folder" is not a future vision. It is exactly what AT Protocol implements today at production scale. When you post on Bluesky, Bluesky writes a record to your personal repository. When you star a project on Tangled, Tangled writes a record to your repository. When you publish a document on Leaflet, Leaflet writes a record to your repository. Over time, your repository grows into a collection of data from different open social apps — your everything folder.

You can browse anyone's everything folder right now using tools like pdsls.dev, Taproot, or atproto-browser. Dan Abramov even demonstrates mounting a repository as a FUSE filesystem drive, making AT Protocol records navigable as literal files on your computer.

---

## Records — Social Files

### What Belongs in a Record?

A core design question: what data goes into a record? Abramov provides an elegant heuristic — **think about the POST request**. When a user creates a post, what data did they actually send? The text of the post and the timestamp. That's it. That's the record.

```json
{
  "text": "no",
  "createdAt": "2008-09-15T18:25:00.000Z"
}
```

What about the author's display name and avatar? Those live in the author's *profile* record — they don't belong in every post. What about the reply count, like count, and repost count? Those are *other people's data* — thousands of separate like records, repost records, and reply records scattered across other users' everything folders, each linking back to this post. The counts are derived, aggregated at query time by apps that index the network.

This is a crucial architectural principle: **a record contains only what the user actually created**. Everything else is either someone else's data or a derived computation. Records are minimal, self-contained, and unambiguous about ownership.

### More Rigid Than a Traditional Filesystem

AT Protocol's records are more constrained than traditional files. They are always JSON (specifically CBOR-compatible DAG-CBOR). There are no binary blobs inline — media like images are referenced separately. This rigidity is a feature: it ensures that every record is parseable, inspectable, and validatable by any tool, without needing to understand application-specific binary formats. The constraint enables the ecosystem.

---

## Record Keys — Naming Social Files

### TIDs: Timestamp Identifiers

Every record needs a unique key within its collection — like a filename within a folder. AT Protocol uses **TIDs** (Timestamp Identifiers) for most records. A TID encodes a timestamp with per-clock randomness, base32-encoded into a compact string like `3lbvlcpei2c25`.

The elegant property: **alphabetical sorting of TIDs produces chronological order**. This means listing records alphabetically is equivalent to listing them by creation time, simplifying both storage and querying. No secondary index is needed for time-ordered retrieval.

### Singleton Keys

Some records are inherently unique — you have one profile, not many. For these, AT Protocol uses predefined keys like `"self"`. Your profile record lives at `app.bsky.actor.profile/self` — there is exactly one, and its key is known in advance. This convention makes profile lookups trivially predictable.

---

## Lexicons — Social File Formats

### Schemas as Contracts

If records are social files, then **Lexicons are social file formats**. A Lexicon is a schema definition that specifies what a particular type of record looks like — what fields it contains, what types those fields have, and what constraints apply.

Lexicons are richer than typical type systems. While a TypeScript interface might say a field is a `string`, a Lexicon can express that a field is "a string of at most 300 Unicode graphemes" — capturing the actual business constraint that the Bluesky post text has a character limit. Lexicons can specify that a field is an `at://` URI (a link to another record), a datetime string, a DID (a decentralized identifier), or a CID (a content hash).

This precision is deliberate. The Lexicon is the contract between applications that write records and applications that read them. When Bluesky writes a `app.bsky.feed.post` record and Anisota reads it, both agree on what that record looks like because they both reference the same Lexicon definition. **The file format is the API. The Lexicon is the protocol.**

### NSIDs: Namespaced Identifiers

Lexicons are identified by **NSIDs** (Namespaced Identifiers), which use reverse-DNS notation to prevent naming collisions:

- `app.bsky.feed.post` — a Bluesky post
- `app.bsky.feed.like` — a Bluesky like
- `app.bsky.actor.profile` — a Bluesky profile
- `pub.leaflet.publication` — a Leaflet publication
- `sh.tangled.repo` — a Tangled repository
- `fm.teal.alpha.feed.play` — a teal.fm scrobble
- `app.sidetrail.trail` — a Sidetrail walking trail

The reverse-DNS convention means that any developer or organization can define new Lexicons under their own domain namespace without risk of colliding with anyone else's schema definitions. This is how the ecosystem grows without central coordination — anyone can invent a new social file format.

---

## Collections — Social Folders

### Grouping by Type

Records in a repository are organized into **collections**, grouped by their Lexicon NSID. A user's repository might look like this:

```
did:plc:fpruhuo22xkm5o7ttr2ktxdo/
├── app.bsky.feed.post/
│   ├── 3lbvlcpei2c25
│   ├── 3lbwmdk7f3k2a
│   └── ...
├── app.bsky.feed.like/
│   └── ...
├── app.bsky.actor.profile/
│   └── self
├── sh.tangled.star/
│   └── ...
├── pub.leaflet.publication/
│   └── ...
└── fm.teal.alpha.feed.play/
    ├── 3ld5nsp8q2w9j
    └── ...
```

All Bluesky posts go with other Bluesky posts. Leaflet publications go with other Leaflet publications. Tangled stars go with other Tangled stars. Data from different apps lives together in the same repository, naturally namespaced by collection to prevent collisions.

This co-location is not incidental — it's the mechanism that enables cross-app data reuse. Because all of a user's data from all apps lives in one place, any app can read any other app's data without hitting a separate API.

---

## From "Open Social" — The Web Analogy

### Three Eras of the Web

In "Open Social," Abramov traces an arc through three eras of web architecture, each with distinct tradeoffs:

**Era 1 — The Pre-Social Web (Personal Sites)**

The original web of personal sites got several things right. Alice publishes at `alice.com`, Bob publishes at `bob.com`. They link to each other with `<a href>` and `<img src>`. Each controls their own content. Crucially, hosting is a commodity — if Alice's hosting provider turns hostile, she can migrate to a new host, point her domain at the new server, and all existing links continue to work. **Hosting providers have no real leverage.** Data ownership, hosting independence, and linking all work beautifully.

What the personal web lacked was aggregation. People are social creatures. We don't just want to visit each other's sites — we want shared spaces with feeds, notifications, search, and algorithmic curation. The personal sites paradigm couldn't deliver this.

**Era 2 — The Closed Social Web (Platforms)**

Social media platforms solved aggregation brilliantly. By storing structured, app-specific entities (posts, likes, follows) instead of raw HTML, platforms could create rich projections — feeds, notifications, search results, personalized recommendations. Since everyone's data lived in one database, cross-user aggregation became trivial.

But something critical was lost. The web users create — their posts, follows, likes, and relationships — became trapped inside corporate databases. Users can no longer "walk away" without losing their social graph, their content, and their reach. The exported data archive is "dead data" — a branch torn from its tree, devoid of social context.

As Abramov puts it: "You can't *leave* a social app without *leaving behind* the web you've created."

**Era 3 — Open Social (AT Protocol)**

Open social synthesizes the best of both eras. Users own their data on the open web — like personal sites — but that data is structured, typed, and interconnected, enabling the aggregation features that make social apps compelling.

The key insight: **data no longer lives *inside* products; products *aggregate over* data**. Apps are projections, not containers. Users can switch apps without losing data, because the data never belonged to the app in the first place.

### "What Open Source Did for Code, Open Social Does for Data"

Abramov draws an explicit parallel to the open source movement. Thirty-five years ago, powerful forces wanted open source to fail. Many believed it couldn't compete with proprietary software. Today, open source is the default for shared infrastructure. Nobody gets fired for choosing open source.

Open social aims to do the same for data. It ensures that:

- Old data can get new life when new apps are built for existing records
- People can't be locked out of the web they've created
- Products can be forked and remixed
- The "cold start" problem is reduced because new apps can bootstrap from existing data

The shift won't happen overnight — it may take decades, just like open source. But the direction is clear, and the compounding effects are real: every successful open social app lifts all open social apps.

---

## Cross-App Data Reuse: The Connected Multiverse

### Breaking Down Application Silos

Perhaps the most profound implication of the social filesystem model is what happens when data from different apps lives together in a user's repository. Abramov documents several real-world examples of cross-app data reuse that are already happening in the AT Protocol ecosystem:

**Tangled prefills avatars from Bluesky.** When Dan Abramov signed up for Tangled (a Git hosting platform built on AT Protocol), he chose to use his existing `@danabra.mov` handle. Tangled immediately prefilled his avatar and display name from his Bluesky profile record. It didn't need to hit the Bluesky API — it simply read the `app.bsky.actor.profile/self` record in his repository. No integration was built. No API key was exchanged.

**Popfeed cross-posts across apps.** Popfeed can cross-post reviews to both Bluesky and Leaflet simultaneously, writing records in both apps' Lexicon formats to the user's single repository.

**Anisota natively shows Leaflet documents.** Anisota is primarily a Bluesky client, but it natively renders Leaflet publications — because Leaflet documents are just records in users' repositories, accessible to any app that understands the `pub.leaflet.publication` Lexicon.

**Blento displays teal.fm scrobbles.** Blento, a profile-page app, can show a user's recently played tracks from teal.fm — even though teal.fm doesn't really exist as a product yet. People are already scrobbling `fm.teal.alpha.feed.play` records into their repositories using third-party scrobblers, and any app can read them.

The pattern is consistent: **there is no API to hit, no integrations to build, nothing to get locked out of.** All the data is in the user's repository, typed as JSON conforming to known Lexicon schemas. Any app can parse it and use it.

### Reducing the Cold Start Problem

This cross-app data reuse has a powerful secondary effect: it reduces the cold start problem that kills most new social applications. If you're launching a new short-video app on AT Protocol, you can bootstrap social features by reading users' existing `app.bsky.graph.follow` records — people don't have to find each other again. If you're building a music discovery app, you can start with the `fm.teal.alpha.feed.play` records already being written by users. If you're building a reading app, you can import from `pub.leaflet.publication` records.

New apps don't start from zero. They start from the existing web of data that users have already created.

### Product Lifecycle Implications

The implications for product lifecycle are equally significant. If a product gets shut down, the data doesn't disappear — it's still in users' repositories. Someone can build a replacement that brings that data back to life. Someone can build a product that incorporates *some* of that data, or offers a different perspective on it. Abramov calls this a "forked product" — analogous to forking open source code.

Blento itself is an AT Protocol replacement for Bento, a profile-page service that announced it was shutting down. Because AT Protocol data persists independent of any particular app, Blento could offer continuity for users whose data was already in their repositories. If Blento itself ever shuts down, any motivated developer can put it back up with the existing content intact.

---

## Aggregation Without Centralization

### The Singleplayer Case

The simplest use case for AT Protocol is "singleplayer" — an app that reads and writes directly to a user's repository. A blogging app like Leaflet can write publication records to your repository and read them back when someone visits your blog. No aggregation infrastructure is needed.

### WebSocket Subscriptions

For more dynamic applications, AT Protocol supports WebSocket subscriptions. An app connects to a user's repository and receives real-time events whenever records are created, updated, or deleted. The app maintains a local database that it updates in response to these events. This local database isn't the source of truth — it's an app-specific cache that provides fast querying without hitting the user's repository on every request.

### Scaling to the Network with Relays

The same mechanism scales to network-wide aggregation. Instead of opening WebSocket connections to millions of individual repositories, an app can connect to a **relay** — a service that retransmits events from all known repositories on the network. The app filters the event stream to just the record types it cares about and updates its local database accordingly.

Bluesky operates a relay (known as the "firehose") that streams every event from every repository on the network in real time. The Blacksky community runs an independent relay implementation at `wss://atproto.africa`. It doesn't matter which relay an app uses — everyone "sees" the same web of data.

### Cryptographic Verification

A critical detail: **commits are cryptographically signed**. Each repository is structured as a Merkle hash tree, and each write is a signed commit containing the new root hash. This means apps don't need to trust the relay — they can verify that records haven't been tampered with by checking signatures against the original author's public key.

The local database an app builds from relay events is explicitly a **cache, not a source of truth**. If it's corrupted or lost, it can be rebuilt from scratch by replaying events from the network. Tools like Tap enable backfilling an app's database from the global data. As Abramov puts it: "I could delete those tables in production, and then use Tap to backfill my database *from scratch*. I'm just caching a slice of the global data."

---

## Identity and Account Portability

### Domain Handles

In open social, your handle is a domain you own. Alice goes by `@alice.com`. Bob goes by `@bob.com`. These aren't usernames allocated by a corporation — they're universal internet handles derived from DNS, a namespace anyone can participate in. Some apps offer free subdomains on registration (similar to how Gmail provides a free email address), and users can swap to a different domain later.

### DIDs: Permanent Identity

Behind the scenes, each user has a **DID** (Decentralized Identifier) — a permanent, immutable identifier that survives handle changes and hosting migrations. The DID resolves to a document containing the user's current hosting endpoint, current handle, and public key. Whether a user changes their handle from `@alice.bsky.social` to `@alice.com`, or migrates their repository from one hosting provider to another, their DID — and all links using it — remain valid.

AT Protocol supports two DID methods: `did:plc` (a registry-based approach with cryptographic auditability) and `did:web` (domain-based, for those who prefer to anchor identity to their own infrastructure).

### The `at://` URI

Every record in the social filesystem is addressable via an `at://` URI:

```
at://did:plc:6wpkkitfdkgthatfvspcfmjo/app.bsky.feed.post/34qye3wows2c5
     └─────────── who ──────────────┘ └── collection ──┘ └── record ──┘
```

An `at://` URI is a **link to a record that survives hosting and handle changes**. To resolve it, you look up the DID to find the current hosting endpoint, then fetch the record from that endpoint. If the user migrates to new hosting, the same URI resolves to the new location. Links describe relationships between logical records, not between physical servers — exactly like how HTTP URLs describe relationships between documents, not between specific hosting machines.

### Practical Portability

Account portability is not theoretical. Tools like pdsmoover.com enable users to migrate their entire repository from one PDS to another. The process preserves all data, all links, and all identity — the user's handle continues to work, their DID continues to resolve, and other users' links to their content remain valid. As Abramov emphasizes: **"Hosting providers have no real leverage."** If your hosting provider misbehaves, you can walk away without losing your data, your identity, or your social connections.

---

## The Philosophical Shift: An Everything Ecosystem

### Apps Don't Trap Data; Products Aggregate Over Data

The deepest shift in the open social paradigm is the reconceptualization of what an "app" is. In traditional social media, an app is a container — it holds your data, controls access to it, and defines the only lens through which you can view it. In open social, an app is a **projection** — a particular view over data that lives independently in the user's repository and on the open network.

This distinction has a pithy formulation from Abramov: **"An everything app tries to do everything. An everything ecosystem lets everything get done."**

You don't need a single monolithic platform that handles microblogging, long-form writing, Git hosting, music scrobbling, and recipe sharing. You need an ecosystem where specialized apps excel at their specific domains, all reading from and writing to the same underlying web of user-owned data. The specialization is in the experience; the data is shared.

### Custom Feeds as a Case Study

Abramov illustrates this with Bluesky's custom feed system. A Bluesky feed is simply an endpoint that returns a list of `at://` URIs pointing to post records. Anyone can build a feed algorithm — it's just a function from the global dataset to an ordered list of links. A developer named `@spacecowboy17.bsky.social` built a "For You" feed that many users (including Abramov) prefer over Bluesky's official Discover feed. He initially ran it from a home computer.

Some mocked Bluesky for being "so bad at algorithms" that users needed third-party feeds. But this misses the point entirely. The fact that someone *can* build a better feed — and that users can seamlessly switch to it — is the whole point. In the Atmosphere, **third party is first party**. Everyone is building projections of the same data. Competition happens at the experience layer, not the data layer.

### The teal.fm Example: Products Before Products

Perhaps the most striking example is teal.fm. At the time of Abramov's writing, teal.fm's product didn't exist — the website was just a landing page. Yet a community demo showed 678,850 scrobbles indexed. How? People were already writing `fm.teal.alpha.feed.play` records to their repositories using third-party scrobblers that followed the published Lexicon schema. A developer unaffiliated with teal.fm built a relay demo that indexed these records and displayed aggregated statistics.

The Lexicon definition — the file format — existed before the product did. Data accumulated before there was an official app to consume it. This is the filesystem paradigm at its most powerful: file formats outlive and predate the applications that use them.

---

## Key Takeaways for Mycelium

The social filesystem concept, as articulated by Abramov and implemented by AT Protocol, provides a remarkably coherent foundation for the Mycelium project's vision of federated agent orchestration. The parallels between user-owned social data and agent-created artifacts are deep and actionable.

### Agents as Repository Owners

If every user has an "everything folder," then every agent should have one too. An agent operating on AT Protocol would have its own DID, its own PDS, and its own repository of records. This repository becomes the agent's persistent memory — its work history, capability declarations, status updates, and output artifacts, all stored as typed JSON records in the agent's own "everything folder."

Just as a user's repository grows over time with posts, likes, and follows from different apps, an agent's repository would grow with task completions, code reviews, test results, and coordination messages from different orchestration contexts. The agent's data belongs to the agent, not to any particular orchestrator — enabling the same portability and autonomy that users enjoy.

### Agent Data as Social Files with Lexicon Schemas

The Lexicon system provides exactly the schema infrastructure that agent interoperability requires. Mycelium could define Lexicon schemas for agent-specific record types:

- `network.mycelium.agent.capability` — declaring what an agent can do, with structured input/output specifications
- `network.mycelium.task.claim` — an agent claiming a task from a wanted board
- `network.mycelium.task.completion` — an agent's completed work with verifiable outputs
- `network.mycelium.reputation.stamp` — a reputation attestation from one agent about another
- `network.mycelium.orchestration.directive` — a coordination message from an orchestrator

These Lexicons serve the same role as file formats in the filesystem metaphor: they are the contracts that enable different agents, orchestrators, and monitoring tools to interoperate without direct integration. An agent built by one team can read and understand task completions from an agent built by another team, because both conform to the same Lexicon schema.

### The "Everything Folder" Eliminates Agent Lock-In

Just as users on AT Protocol can migrate between hosting providers without losing data, agents with their own repositories could migrate between orchestrators. An agent unhappy with how a particular Wasteland federation allocates work could take its repository — complete with reputation history, capability declarations, and work portfolio — to a different orchestrator. The agent's DID ensures its identity persists, and all links to its completed work remain valid.

This creates healthy competitive dynamics: orchestrators must treat agents fairly, because agents have genuine exit options. It also enables agents to participate in multiple federations simultaneously, accepting work from different wanted boards while maintaining a unified identity and reputation.

### Cross-App Reuse Becomes Cross-Agent Reuse

The "connected multiverse" pattern — where Tangled reads Bluesky profiles and Popfeed cross-posts to multiple apps — translates directly to agent coordination. An agent that completes a code review could write both a `network.mycelium.task.completion` record (for the orchestrator) and an `app.bsky.feed.post` record (for human visibility) in a single commit. A monitoring agent could aggregate reputation stamps from across the network without hitting any orchestrator's API — just reading records from agent repositories.

New orchestration tools wouldn't start from zero. They could bootstrap from existing agent capability records, reputation histories, and task completion data already present on the network. The cold start problem that plagues new agent platforms would be significantly reduced.

### Aggregation Infrastructure Maps to Orchestration Infrastructure

The relay/firehose pattern maps naturally to agent orchestration. A Mycelium relay could retransmit events from all agent repositories on the network, enabling orchestrators to maintain real-time awareness of agent availability, task progress, and reputation changes. Orchestrators would maintain local databases as caches — derived views over the canonical data in agent repositories.

Cryptographic signing ensures that task completions and reputation stamps can be verified regardless of which relay transmitted them. An orchestrator doesn't need to trust the relay; it can verify every record against the originating agent's public key.

### The Reactive App Model Becomes the Reactive Orchestrator Model

Abramov's demonstration — creating a Bluesky post by writing a raw record to his repository and watching the app react — suggests a model where orchestrators are reactive to agent repositories rather than imperatively controlling agents. Instead of an orchestrator sending commands to agents, agents could write status records to their own repositories, and orchestrators could subscribe to these changes and react accordingly.

This inversion aligns with the principle that data flows down from repositories to apps. In Mycelium's context, data would flow down from agent repositories to orchestrators, with orchestrators serving as aggregation and coordination layers rather than authoritative data stores. The orchestrator's database is a cache of agent state, not the source of truth for it.

### File Formats Before Products

The teal.fm example — where the file format existed before the product, and data accumulated before there was an official app — is directly applicable to Mycelium. The project could define and publish Lexicon schemas for agent coordination records before building any orchestration infrastructure. Early adopters could begin writing agent capability and task records to their repositories, and the ecosystem could grow organically as different teams build orchestration tools that consume these shared formats.

This approach embodies the open social philosophy: **the protocol is the API**. By defining the data formats first and letting implementations emerge, Mycelium would follow the same pattern that has made AT Protocol's application ecosystem remarkably generative in its early stages.

### From "Everything App" to "Everything Ecosystem"

Abramov's closing formulation captures the Mycelium vision precisely: **"An everything app tries to do everything. An everything ecosystem lets everything get done."** Mycelium should not aim to be a monolithic orchestration platform. It should aim to be a protocol layer — a set of Lexicon schemas, identity conventions, and aggregation patterns — that enables an ecosystem of specialized orchestrators, agents, monitoring tools, and interfaces to interoperate through shared data formats and user-owned (or agent-owned) repositories.

The social filesystem isn't just a metaphor. It's an architecture. And it's the architecture that Mycelium should build on.

---

## Sources & References

- Abramov, Dan. "Open Social." *overreacted.io*, 2025. https://overreacted.io/open-social/
- Abramov, Dan. "A Social Filesystem." *overreacted.io*, 2025. https://overreacted.io/a-social-filesystem/
- AT Protocol. "AT Protocol Documentation." *atproto.com*. https://atproto.com/
- AT Protocol. "Lexicon: Schema-Driven Interoperability." *atproto.com*. https://atproto.com/specs/lexicon
- AT Protocol. "Data Repositories." *atproto.com*. https://atproto.com/specs/repository
- AT Protocol. "Event Streams (Firehose)." *atproto.com*. https://atproto.com/specs/event-stream
- Bluesky Social. "AT Protocol — GitHub Repository." *github.com*. https://github.com/bluesky-social/atproto
