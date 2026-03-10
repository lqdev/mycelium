# Deep Dive 05: Muni.town, Local-First Computing, and the Architecture of Village-Scale Resilience

*Mycelium Research Series — Report 5*
*Investigating federated agent orchestration on decentralized social protocols*

---

## Executive Summary

This report examines Muni.town's vision for decentralized community infrastructure through three interconnected lenses: the Leaf server architecture powering Roomy, the philosophy of village-scale resilience that motivates the project, and the broader paradigm of atmospheric computing that situates it within the AT Protocol ecosystem. Together, these threads weave a compelling narrative about what digital infrastructure should look like — not just for privileged westerners with uninterrupted broadband, but for Palestinian refugee camps, for Ugandans enduring election-related internet shutdowns, and for anyone who recognizes that our digital infrastructure is, as Muni.town founder Erlend Sogge Heggen puts it, "all hanging by a thread."

For the Mycelium project, this report surfaces critical architectural patterns: event-sourced data management with application-agnostic module systems, the practical tension between local-first ideals and cloud-first commercial realities, and the emerging model of data-banking cooperatives as people-governed alternatives to corporate data silos. Each of these has direct implications for how we design federated agent infrastructure that can operate offline, in mesh networks, and under community governance.

---

## Roomy and Muni.town: Community Gardens for the Atmosphere

### What Roomy Is

Roomy is a digital community platform designed as an alternative to Discord and Facebook Groups — what Muni.town describes as a "digital community garden for knowledge cultivation." Unlike the extractive attention-economy platforms it seeks to replace, Roomy is open software connected to the Atmosphere, the community name for the AT Protocol ecosystem.

The project has been through a long gestation. As Heggen acknowledges in the village-scale resilience blog post:

> Roomy has taken a long time to *get right*. Long enough that some people lost faith in us along the way, and that's fair. We'll try our best to earn it back. A year ago we already had something that worked. Roughly every two months since then we have scrapped what we had and started anew.

This iterative process — building, scrapping, keeping the best bits, compounding — has been architecture-oriented rather than solutions-oriented. The partnership with ATmosphereConf marks the transition point: from pure architectural exploration to production continuity with real community relationships to maintain. No more do-overs. A real, pro-bono customer relationship. The first of many.

### Positioning in the Ecosystem

Roomy occupies a specific niche in the AT Protocol ecosystem. While Bluesky provides "networked feeds for mobilization," Roomy provides "intertwined spaces for organisation." This is a deliberate strategic complement — mobilization and organization are the two essential modes of digital power for communities and movements.

The platform is designed to be community-first rather than security-first. Heggen draws a careful distinction: "Your local libraries and community gardens optimize for accessibility, not defensibility. So too with the type of space-making Roomy exists to facilitate." Briar is the medicinal treatment for a sick democracy; Roomy is preventive care. Both are needed, and they are by no means mutually exclusive.

---

## The Leaf Server Architecture

### Three Components

Roomy's architecture consists of three major components:

1. **The Roomy Web App** — the user-facing interface
2. **The AT Protocol PDS** — storing user identity, authentication, and public uploads
3. **The Leaf Server** — providing storage and real-time sync for space data

The critical architectural decision is that **Leaf is application-agnostic**. It contains no Roomy-specific code whatsoever. Instead, it hosts user-defined "modules" that customize how the server handles data for a particular "stream." As the Leaf blog post explains, it functions as "an opinionated kind of PaaS designed specifically for Event Sourcing."

This separation is powerful: modules can be upgraded by the app without requiring infrastructure changes. New features can be added and experiments tried without keeping server deployment and app in sync. The server is lightweight and easy to self-host.

### Streams: The Multi-Player PDS

Almost all data on the Leaf server exists in streams — analogous to repos in AT Protocol, but with a crucial difference. Unlike user repos in AT Protocol which belong to individuals, Leaf streams are often for *communities*. In Roomy, streams represent spaces (similar to Discord guilds) with controlled write and read access from multiple user accounts.

As fellow Roomy developer @meri.garden described it: **"Leaf as a multi-player PDS."**

Stream identifiers use DIDs, just like AT Protocol accounts. When a new stream is created, it generates a new DID and publishes it to plc.directory. The stream creator can add additional rotation keys to the DID that enable migration to a different Leaf server — critical for maintaining user sovereignty if a hosting provider goes rogue.

### Events as Fundamental Data Structure

All data in a stream is represented by **events**, each with a deliberately minimal data model:

- **Index** — a monotonically incrementing number for ordering
- **User** — the DID of the account that submitted the event
- **Payload** — a binary array containing the event data

The payload is format-agnostic: it could be JSON, CBOR, or any binary or text format. This design gives applications complete flexibility to optimize encoding for their specific needs.

Each stream maintains its events in a **separate SQLite database**. This per-stream isolation provides natural data boundaries and simplifies operations like migration, backup, and cleanup.

### The Module System

Modules are the customizable logic layer of streams, responsible for three concerns:

**Authorization.** Before any event is saved to a stream, it must be authorized by the stream's module. Read authorization works similarly — the only way to retrieve events is through queries defined by the module. A simple SQL authorizer might look like:

```sql
select unauthorized("Only stream owner can add events")
    where (select creator from stream_info) != (select user from event);
```

**Materialization.** For every event that passes write authorization, the module may run a "materializer" that caches information about the event in its own SQLite database. This database can index events based on application-specific criteria — which channel a chat message was sent in, what permissions exist, full-text search indices, statistics like when a channel was last updated.

**Aggregation.** The module's database naturally enables server-side aggregations over events. Full-text search, activity statistics, permission lookups — all computed and cached on the server, available through defined queries for clients.

Modules are currently written entirely in SQL, with access to custom SQL functions defined specifically for Leaf. The team is considering extending this with WASM-defined custom functions or lightweight scripting languages for logic that is impractical in pure SQL.

### Subscriptions and Real-Time Updates

Queries defined by modules can be **subscribed to**. When a new event arrives in a stream, subscribed queries are re-run and push updates to clients if results change. This enables real-time collaborative features where multiple users see updates flow instantly.

The architecture also supports **ephemeral events** — events that are not persisted to the stream database. These serve use cases like typing indicators and online presence. Ephemeral events can have their own materializers that store only the latest state (e.g., the last message a user read) without accumulating event history.

### Why Off-Protocol?

A natural question arises: if all the data lives on the Leaf server, how does AT Protocol factor in? The honest answer is that Roomy's data is "off-protocol" — and this is a deliberate, well-reasoned choice driven by several practical constraints:

- **Private data.** All data on the PDS is public. Chat messages in private channels cannot be stored without encryption, and publicly distributing encrypted data is poor practice.
- **Many, tiny, real-time events.** The PDS has rate limits on writes, and the firehose lacks the latency characteristics needed for chat synchronization.
- **Notifications for private data.** Even with private data storage solved, the firehose-based notification model doesn't work for private content.
- **Local-first synchronization.** Depending on AT Protocol as the canonical data source makes offline-first operation much harder to achieve.

AT Protocol remains essential for authentication (providing a "Login with Google"-style experience backed by self-hostable identity) and for interoperability with other Atmosphere apps like Bluesky, Smoke Signal, Streamplace, and Semble. Public uploads are stored on the PDS, and the long-term plan includes continuous backups of public Roomy data to the user's PDS.

The Leaf server provides the same data ownership properties as the PDS — self-hostable, migratable between providers, accessible via API — just with a different interface optimized for community-scale, event-sourced, real-time data.

---

## Village-Scale Resilience

### The Rashidieh Story

The philosophical foundation of Roomy extends far beyond Western tech-sector concerns about platform lock-in. It begins in Rashidieh, a primarily Palestinian refugee camp in southern Lebanon housing over 30,000 people — the same size as Molde, the biggest city near Heggen's hometown in Norway.

> One of the most extraordinary things about communities forced to persevere in extreme living conditions is that they develop bespoke technologies strictly for on-the-ground problems. Out of pure necessity their technological sophistication will in some cases far exceed the stagnant status quo of the so-called Developed World.

Rashidieh is more connected than one might assume. Many residents have mobile phones; some houses have internet-connected wifi routers. But connectivity remains a luxury. Village-wide disconnections can result from accidental power outages or intentional disruptions by oppressive authorities.

### Internet-in-a-Box

The practical response to these conditions was embodied by one resident — a "friendly neighborhood supergeek" who was regularly downloading the Internet-in-a-Box and serving it back out to his local community via residential wifi. When residents lost access to the outside world, locals could still access the most recent download of essential internet content through his home network.

This is the seed of what Muni.town calls the **Community-in-a-Box** vision: given a robustly designed toolkit, residents of places like Rashidieh could autonomously maintain a local-area communications network independent of higher authority. When connectivity drops, they would be out of reach to the outside world but not to each other.

### The Western Illusion of Permanence

The report's most striking rhetorical move is turning these concerns back on Western audiences:

> You know what happens if I lose internet connectivity here in the fancy capital of Norway, Oslo? I'm completely f\*\*\*ed is what happens. Local-area redundancy networks and backup caches? We have no such thing. *No need! We'll always be connected, we're a Developed Country.*

Yet Portugal experienced a near nation-wide outage. Even with satellite phones as fallback, the communications grid was heavily overburdened and unreliable during that "inconceivable crisis." The closer you examine our digital infrastructure, the more apparent its fragility becomes. As Heggen writes: "it only takes Vlad and Daffodil on opposite sides of the earth to both get sick on the same day for the whole grid to come crashing down."

The fundamental argument is that uninterrupted high-speed connectivity is contingent on an "energy-blind, self-terminating economy of hyperconsumption" — infrastructure premised on perpetual growth that has pushed past planetary boundaries while transferring wealth upward. Every essential layer of digital infrastructure is "chiefly owned, operated and controlled by a stack of oligarchic companies in varying degrees of partnership with the state they're holding hostage."

---

## Local-First Computing in Practice

### The Landscape of Resilient Communication

The practical landscape of local-first, resilience-oriented communication technology includes:

**Briar** — In Iran, Briar is keeping people connected during internet shutdowns when mainstream alternatives fail. It operates over Bluetooth, WiFi, and Tor, enabling peer-to-peer encrypted communication that doesn't depend on centralized infrastructure.

**Bitchat** — A "vibe-coded but demonstrably good enough" application that has topped app store charts in Uganda during turbulent elections and in Jamaica during Hurricane Melissa. These are not theoretical use cases — they are real deployments where local-first communication became a matter of survival.

These tools represent "low-bandwidth, intermittent, lo-fi tech that's peer-to-peer connected, distributed via bluetooth mesh networks and pirate wifi antennas" — technology that "appears utterly irrelevant until it's suddenly a matter of life and death."

### The Curb-Cut Effect

Muni.town frames the relationship between marginalized community needs and universal benefit through the **curb-cut effect** — the observation that designing for the needs of marginalized populations benefits everyone. Curb cuts were designed for wheelchair users but benefit parents with strollers, delivery workers with carts, travelers with suitcases, and countless others.

> The necessary affordances of a liberatory technology are largely the same no matter what part of the planet you're living on.

Technology that works for a refugee camp with intermittent connectivity also works for a Norwegian city during an infrastructure outage, for a rural American community with spotty broadband, or for anyone who simply wants their communications to work regardless of what happens to their internet provider.

### Roomy's Pragmatic Path

Roomy was designed for local-first operation but is being built cloud-first — a deliberate pragmatic choice. The project's history includes attempts to use Willow (not ready yet), Keyhive & Beelay (also not ready), and Jazz (performance problems and missing features).

The current event-sourcing architecture explicitly preserves the path toward local-first:

> The event sourcing model we are using allows you to make edits offline and later sync those edits to a server, to give us local-first features. Similar to Git, we can hypothetically have temporary "forks" of a chat space that allow you to chat with someone that you have a LAN connection with, even if you are offline, before later "pushing" those changes to the main "branch."

The team's philosophy on this tension is characteristically honest:

> The best way to build a local-first app is to first build any kind of app commercially successful enough to sustain its ongoing development.

Cloud-independence has already been achieved through participation in the Atmospheric Computing movement. Full local-first operation is the directional goal, not the launch requirement.

---

## Atmospheric Computing

### Paul Frazee's Paradigm

Paul Frazee, an engineer at Bluesky, articulated the paradigm of "atmospheric computing" — a framework that positions AT Protocol not merely as a social networking protocol but as a fundamental shift in how cloud computing operates.

The diagnosis is clear: **cloud computing won.** It is convenient, ubiquitous, and scalable. But from the personal computing perspective, "the cloud has been a disaster." Control over identity, friends, platforms, privacy — all surrendered to someone else's better computer. The economic consequences are equally stark: "somehow the liberation technology of computing seemed to liberate seven specific companies and nobody else."

### The XMPP and Google Reader Lesson

Frazee identifies a critical historical turning point: the death of XMPP and Google Reader. These represented moments when large companies with large clouds realized they didn't have to share networks anymore. They could build proprietary networks that lived entirely within their clouds and never interacted with competitors.

This created a structural trap: running a personal cloud became pointless because it couldn't connect to anyone. The only viable strategy was building a successful app so people join *your* big cloud — perpetuating the very problem you set out to solve.

### The Solution: Bridge the Clouds

The atmospheric computing paradigm resolves this by enabling **connected clouds**:

> Atmospheric computing is a paradigm of connected clouds. We use "The Atmosphere" to describe the AT Protocol's network. As the Atmosphere is the thing that clouds float within, the metaphor feels apt.

Connected clouds preserve the always-on convenience of cloud computing while restoring the personal computing values of data ownership and program control. The main benefit is **interoperation**: a Bluesky account works on Leaflet, Tangled, and any other Atmospheric application. Apps don't need to talk directly to each other — they all talk to users' account hosts.

This enables several properties:

- **Cooperative computing** — third-party services can present as first-party. The most popular algorithm on Bluesky is "For You," run by a community member on his gaming PC.
- **Cold start resolution** — new apps inherit the entire user base of the Atmosphere by design.
- **Unwalled gardens** — self-hosted instances see all the same users and activity.
- **Personal clouds that matter** — because interoperation means personal infrastructure can connect to everyone else's.

### The Technical Architecture

Frazee describes the underlying architecture with remarkable clarity. The system uses eventual consistency with canonical databases in a single server role (the user's personal data store). It employs a document-store model (JSON) for natural colocation of related data. A CDC (Change Data Capture) log enables consumption by any subscriber. Events are cryptographically signed so the log can be gossiped across organizational boundaries without authenticity callbacks to the origin.

The network is sharded by user — each user is one database with strict serial ordering internally but only causal ordering between databases. Applications subscribe to users' CDC logs and feed them into event handlers, building aggregated views. A schema description language (Lexicon) enables agreement on record schemas and API contracts.

As Frazee summarizes: "That's what AT Protocol is."

---

## Data-Banking Cooperatives and Digital Homeownership

### Personal Data Storage as Credit Unions

Muni.town's most radical proposition may be its framing of personal data storage through the lens of banking cooperatives. The argument proceeds from a simple observation: in the PDS paradigm, most people's data won't move from the cloud to their personal computer. Most people will still rely on institutional cloud services. But instead of data-banking with shareholder-controlled corporations, data can be entrusted to the equivalent of **member-owned credit unions for data storage**.

The parallel is compelling: one in every three US adults banks with a Credit Union. Achieving similar or better numbers for data storage "is far from inconceivable considering how much our collective experience with Big Banking mirrors that of Big Tech/Social."

### Existing Models

The concept of data cooperatives has already gained traction in the fediverse:

- **social.coop** — a cooperatively owned Mastodon instance
- **data.coop** — a Danish data cooperative
- **cosocial.ca** — a Canadian cooperative social media provider

In the AT Protocol network, similar institutions are emerging:

- **Northsky** — a forthcoming cooperatively-owned AT Protocol hosting provider
- **Blacksky** — a community-oriented PDS host
- **Eurosky** — a forthcoming European AT Protocol host

Whether these providers are formal cooperatives in the legal sense matters less than their operational principles: "any sufficiently transparent, democratic and community-oriented data bank is a valid steward and co-creator of an Open Social."

### Shifting the Conversation

The most potent framing in the Personal Data Storage essay concerns how data ownership discourse changes when infrastructure changes:

> Data Ownership as a conversation changes when data resides primarily with people-governed institutions rather than corporations. Rather than arguing for what kinds of data we *ought to be* able to download from the corporate silos, the platforms should be asking us what kinds of data they may copy from *our servers*, and only with strictly temporary allowances.

This inverts the entire power dynamic. Instead of users petitioning platforms for data portability, platforms must petition users for data access.

### Tim Berners-Lee's Solid Vision

The personal data storage concept has deep roots. Tim Berners-Lee drafted a specification for "Socially Aware Cloud Storage" in 2009 and has continuously championed the vision through the Solid Protocol:

> We have the technical capability to give that power back to the individual. Apps running on Solid don't implicitly own your data — they have to request it from you and you choose whether to agree, or not. Rather than being in countless separate places on the internet in the hands of whomever it had been resold to, your data is in one place, controlled by you.

While Solid has produced an official W3C web specification, it has not yet achieved mainstream adoption. Its primary financial sponsor Inrupt has focused on the enterprise market. But as Muni.town notes, the concept has transcended any single protocol:

> Personal Data Storage has long since escaped containment as a concept pertaining to any specific protocol. Some implementations of it will be more mainstream than others, but pragmatic data coops can be protocol-agnostic and storage formats are transmutable.

### Digital Homeownership

Muni.town frames the PDS through the metaphor of **digital homeownership**. Most people's home on the internet is situated in a "hyper-scale condominium" like `facebook.com/yourhome` — rented in exchange for data and attention, which platforms use to sell advertising and manufacture consent.

The alternative: own your digital presence. A domain is digital real estate; a website is the house on top. Agency on the web begins with a personal website on your own domain. Rather than renting from corporations, users can own their digital presence outright.

This isn't merely philosophical. If you self-host your account and your site, "you can get kicked off every app in the Atmosphere but you've still got that identity and that presence." Even from the perspective of someone who works at Bluesky and believes in good moderation, Frazee considers this kind of check on social apps' power to be a healthy design property.

---

## The Secure-vs-Accessible Spectrum

Muni.town draws a thoughtful distinction between different modalities of resilient communication technology:

**Briar** represents security-first design — encrypted, anonymous, built for hostile environments where surveillance is an immediate physical threat. It is "part of the medicinal treatment of a sick democracy."

**Roomy** represents community-first design — accessible, inviting, built for cultivating the connective tissue of healthy communities. It is "preventive care."

Both are needed. As Heggen observes: "businesses want intranets and nerds wanna hack on cryptographic locks, leaving scarcely any money and attention left for cozy open-access spaces on the web." Roomy addresses this gap — the space between hardened security tools and extractive commercial platforms where communities can actually gather, organize, and cultivate shared knowledge.

The distinction matters for Mycelium: agent infrastructure similarly needs both hardened security for adversarial environments and accessible, community-oriented tooling for everyday collaborative orchestration.

---

## Key Takeaways for Mycelium

### 1. The Leaf Architecture as a Blueprint for Agent Data Management

The Leaf server's architecture maps remarkably well to federated agent infrastructure requirements:

**Event-sourced streams** are natural containers for agent work logs, decision records, and coordination state. Each agent or agent-group could maintain a Leaf stream recording every action taken, every decision made, and every coordination event exchanged. The append-only, event-sourced model provides the auditability and reproducibility that agent systems desperately need.

**Application-agnostic modules** demonstrate how to build general-purpose infrastructure that serves specialized agent needs. Just as Leaf modules customize authorization, indexing, and aggregation without changing the core server, Mycelium agent modules could customize how work is validated, how reputation is calculated, and how coordination queries are resolved — all without modifying the underlying agent runtime.

**Format-agnostic payloads** align with the reality that different agent types will use different serialization formats. A code-generation agent might produce structured JSON work artifacts; a document-processing agent might work with binary formats; a coordination agent might use compact CBOR messages. The infrastructure should be indifferent to encoding.

**Per-stream SQLite databases** offer a model for agent data isolation. Each agent workspace, each coordination channel, each reputation ledger could maintain its own isolated database — simple to reason about, straightforward to migrate, and naturally bounded in scope.

### 2. Local-First Agent Operations

The village-scale resilience philosophy has direct implications for agent infrastructure:

**Offline-capable agents.** Agents must be able to operate without continuous connectivity to centralized orchestrators. The event-sourcing model enables this: agents can accumulate work events locally and sync when connectivity is restored, analogous to Roomy's planned Git-like "fork and push" model for offline chat.

**Mesh-network coordination.** In environments with only local-area connectivity (LAN, Bluetooth mesh, pirate wifi), agents need to coordinate peer-to-peer without relying on cloud infrastructure. The Leaf architecture's separation of event production from centralized aggregation means agents can exchange events directly over any transport layer.

**Graceful degradation.** The Briar/Bitchat precedent shows that "good enough" local communication is infinitely more valuable than perfect cloud communication that doesn't work. Mycelium agents should be designed to degrade gracefully — reducing coordination sophistication rather than failing entirely when infrastructure degrades.

**The practical path.** Roomy's pragmatic approach — "the best way to build a local-first app is to first build any kind of app commercially successful enough to sustain its ongoing development" — applies equally to agent infrastructure. Build cloud-first with local-first architecture. Ship something that works. Iterate toward full decentralization.

### 3. Community-Governed Agent Hosting

The data-banking cooperative model suggests a governance framework for federated agent infrastructure:

**Agent hosting cooperatives.** Just as data cooperatives like social.coop provide community-governed hosting for social data, agent hosting cooperatives could provide community-governed compute for agent orchestration. Members collectively own the infrastructure, set policies for agent behavior, and maintain democratic control over how their agents operate.

**The credit union analogy.** If one in three US adults can bank with a credit union, a similar fraction could plausibly host their agents with a cooperative rather than a corporate provider. The economic model is well-understood; only the technical substrate differs.

**Federated but governed.** The Northsky/Blacksky/Eurosky model — multiple independent PDS hosts with different community orientations, all interoperating through shared protocols — maps directly to a vision of multiple agent hosting cooperatives. Each cooperative might specialize (development agents, research agents, creative agents) while sharing a common coordination protocol.

**Data sovereignty for agents.** The principle that "platforms should be asking us what kinds of data they may copy from our servers" extends to agent data. Work logs, learned models, reputation histories — these belong to the agent operator (individual or cooperative), not to the orchestration platform.

### 4. The Multi-Player PDS Concept

The "Leaf as a multi-player PDS" concept opens particularly interesting territory for Mycelium:

**Shared agent workspaces.** Just as Roomy streams enable multiple users to collaborate in shared spaces, Mycelium could implement shared agent workspaces where multiple agents (potentially from different operators) collaborate on shared tasks with controlled read/write access.

**DID-based workspace identity.** Using DIDs for workspace identifiers (as Leaf does for streams) means agent workspaces have portable, self-sovereign identity. A workspace can migrate between hosting providers while maintaining its identity, its membership, and its accumulated history.

**Module-based governance.** Different agent workspaces could load different governance modules — one workspace might require consensus from all participating agents before accepting events, another might delegate authority to a designated orchestrator agent, and a third might implement reputation-weighted voting.

### 5. The Atmospheric Computing Model for Agent Interoperation

Paul Frazee's atmospheric computing paradigm provides the meta-framework for thinking about how federated agent networks should work:

**Personal agent clouds.** Each individual (or organization, or cooperative) runs their own agent infrastructure — their personal cloud. These clouds are connected through open protocols, enabling agents to discover, authenticate, and coordinate across organizational boundaries.

**The XMPP lesson.** Closed agent platforms will inevitably follow the same path as closed social platforms — lock-in, extraction, and the elimination of interoperability once market position is established. Only open protocols prevent this.

**Cooperative computing with untrusted peers.** The AT Protocol's architecture — signed CDC logs, content-hash checksums, gossip-capable event distribution — provides the technical foundation for agents to coordinate across trust boundaries. Agents can verify the authenticity of other agents' work products without trusting the infrastructure that delivered them.

**The sovereignty check.** The ability to self-host means no single entity can unilaterally exclude an agent from the network. This is not just a theoretical property — it is the fundamental architectural guarantee that prevents federated agent networks from degenerating into centralized ones.

### 6. The Curb-Cut Principle Applied to Agent Design

Designing agent infrastructure for the most constrained environments — offline operation, low-bandwidth networks, community governance — produces infrastructure that is more robust and more useful for *everyone*:

- Offline-capable agents work better in data centers with network partitions too
- Mesh-network coordination protocols are also faster for LAN-local agent swarms
- Community governance models are also healthier for corporate agent deployments
- Format-agnostic event systems are also more interoperable across toolchains

The necessary affordances of resilient agent infrastructure are largely the same whether you're orchestrating agents in a well-connected cloud data center or coordinating emergency response agents over a Bluetooth mesh network in a disaster zone.

---

## Conclusion

Muni.town's work represents something rare in the technology landscape: a project that simultaneously pursues ambitious technical architecture (the Leaf server, event-sourced community data management, real-time subscriptions) and deeply considered social philosophy (village-scale resilience, data cooperatives, digital homeownership). Neither dimension is sacrificed for the other.

For Mycelium, the lesson is clear. Federated agent orchestration cannot be built purely as a technical exercise. The architecture must encode values: data sovereignty, community governance, graceful degradation, offline capability, interoperability across trust boundaries. The Leaf server demonstrates that these values can be expressed in concrete architectural decisions — per-stream databases, application-agnostic modules, format-agnostic payloads, DID-based identity with migration keys.

The village-scale resilience philosophy reminds us that the most sophisticated technology is often the technology that works when everything else fails. As we design agent infrastructure for the Wasteland, we would do well to ensure it also works for Rashidieh.

---

## Sources & References

- Heggen, Erlend. "Village Scale Resilience." *Muni.town Blog*, 2024. https://blog.muni.town/village-scale-resilience/
- Heggen, Erlend. "Personal Data Storage Is an Idea Whose Time Has Come." *Muni.town Blog*, 2024. https://blog.muni.town/personal-data-storage-idea/
- Heggen, Erlend. "Digital Homeownership." *Muni.town Blog*. https://blog.muni.town/digital-homeownership/
- Muni.town. "Leaf 0.3: The Server Behind Roomy." *Muni.town Blog*. https://blog.muni.town/leaf-0-3-the-server-behind-roomy/
- Muni.town / Leaf Server. https://leaf.muni.town/ · GitHub: https://github.com/muni-town/leaf
- Roomy. https://roomy.chat/
- Frazee, Paul. "Atmospheric Computing." *pfrazee.com*. https://www.pfrazee.com/blog/atmospheric-computing
- Abramov, Dan. "Open Social." *overreacted.io*. https://overreacted.io/open-social/
- Berners-Lee, Tim. "Socially Aware Cloud Storage." *W3C Design Issues*, 2009. https://www.w3.org/DesignIssues/CloudStorage.html
- AT Protocol Documentation. https://atproto.com/
- Kleppmann, Martin et al. "Local-First Software: You Own Your Data, in Spite of the Cloud." *Ink & Switch*, 2019. https://www.inkandswitch.com/local-first/
- Internet-in-a-Box. https://internet-in-a-box.org/
- Briar Project. https://briarproject.org/
- Solid Protocol. https://solidproject.org/
- MIT IDE. "Data Cooperatives." https://ide.mit.edu/sites/default/files/publications/Data-Cooperatives-final.pdf
