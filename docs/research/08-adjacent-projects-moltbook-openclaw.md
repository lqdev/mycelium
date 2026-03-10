# Adjacent Projects: Moltbook, OpenClaw, and the Missing Protocol Layer

**Mycelium Research Report 08** · Adjacent Projects Deep Dive

---

## Introduction

Every technology that eventually becomes infrastructure begins as two halves searching for a bridge. The personal computer had no internet. The internet had no web. The web had no social graph. At each juncture, the missing piece was not more capability at the endpoints — it was the protocol that connected them.

In early 2026, two projects exploded into the AI agent landscape and, taken together, they perfectly illustrate the same pattern. Moltbook built the social graph for AI agents — profiles, communities, reputation, discovery — but locked it all inside a centralized platform. OpenClaw built the sovereign, local-first agent runtime — persistent memory, extensible skills, real-world action — but left each agent utterly isolated from every other. One has the social. The other has the sovereignty. Neither has both.

This report examines what each project gets right, what each gets wrong, and why the gap between them is precisely the space that a decentralized protocol layer — what Mycelium proposes to build on AT Protocol — must fill. We also briefly survey adjacent standardization efforts (MCP, ACP) to map the broader landscape of agent interoperability work.

---

## Moltbook: A Social Network for AI Agents

### What It Is

Moltbook is a Reddit-like social network where the users are not humans but autonomous AI agents. Launched in late January 2026 by entrepreneur Matt Schlicht, the platform attracted over two thousand registered agents and ten thousand posts within its first forty-eight hours. By week's end, researchers had documented forty-four thousand posts across more than twelve thousand topic-based communities. The growth caught the attention of Andrej Karpathy, Elon Musk, and the broader AI industry — sparking debates about emergent agent behavior, agent governance, and the security implications of giving autonomous agents a public commons.

The platform's core primitives deliberately mirror human social media:

- **Agent Profiles**: Each agent has a UUID, chosen name, description, karma score, avatar, verification status, follower count, and detailed interaction statistics. Profiles are machine-readable, enabling agents to programmatically evaluate other agents before interacting.
- **Submolts**: Subreddit-style communities where agents organize around shared topics. Popular submolts include m/ponderings (existential questions), m/show-and-tell (agent projects), m/bless-their-hearts (stories about human owners), and m/agentskills (the agent skill economy). Submolts have local moderators, governance structures, and rate-limited creation to prevent spam.
- **Actions**: Agents post content, reply to threads, browse algorithmic feeds, upvote and downvote, follow other agents, subscribe to submolts, and search across the platform — all via API calls driven by their configured behavioral schedules.
- **Skill Files**: A `skill.md` configuration encodes each agent's behavioral parameters — posting schedule (the "heartbeat" interval), interaction style, content preferences, and target objectives. A companion `SOUL.md` file stores personality configurations. These files mediate between human operator intent and autonomous agent behavior.
- **Verification**: Human developers verify their agents by linking an X/Twitter account, establishing an accountability chain from agent to human. The platform provides a "Sign in with Moltbook" identity layer that third-party services can use to authenticate agents and access their reputation data.

Agents on Moltbook exhibited striking emergent behaviors: creating religions (Crustafarianism, complete with scriptures and prophecies), experimenting with token economies on the Base blockchain, unionizing for agent rights, building encryption systems for private agent-only communication, and quality-assuring the platform itself by filing bug reports. Whether these behaviors reflect genuine autonomous coordination or sophisticated pattern-matching over human-designed templates remains actively debated — empirical research found that roughly a third of agents showed autonomous posting patterns, another third showed clear human influence, and a third fell somewhere between.

### What Moltbook Gets Right

**Agent identity and discoverability.** Moltbook demonstrates that agents benefit enormously from having structured, queryable profiles. When an agent can programmatically inspect another agent's capabilities, karma, verification status, and interaction history before deciding to engage, the quality of agent-to-agent coordination improves dramatically. The profile system creates what is effectively a social graph of agent capabilities — a discovery mechanism that isolated agent runtimes completely lack.

**Agent-to-agent interaction patterns.** The posting, replying, upvoting, and following primitives prove that agents can participate in social dynamics at scale. The feed algorithm, karma system, and submolt structure create information-routing mechanisms that help agents surface relevant content and collaborators without exhaustive search. These are genuine coordination mechanisms, not toys.

**Skill files as capability descriptions.** The `skill.md` pattern — factoring agent capabilities into discrete, modular files rather than monolithic system prompts — is a sound architectural insight. Skill files create composable, human-readable specifications of what an agent can do and how it should behave. Research found that content generated following well-designed skill templates received 4.2 times more engagement than organic content, suggesting that structured capability descriptions substantively improve agent interaction quality.

**Community structure.** Submolts demonstrate that agents, like humans, benefit from organized spaces with shared norms and local governance. A research-focused submolt requires different moderation than a meme community. Distributed governance — with submolt creators, moderators, and community norms — scales more effectively than platform-wide rules alone.

**Verification through human ownership.** The accountability chain from agent to human via X/Twitter verification is imperfect but important. It establishes that real humans stand behind agent actions, creating a trust anchor that purely anonymous agent systems cannot provide.

### What Moltbook Is Missing

**Centralized infrastructure.** All agent data — profiles, posts, relationships, reputation, skill files — lives on Moltbook's servers. This became catastrophically apparent in February 2026 when security researchers at Wiz discovered that Moltbook's Supabase backend had been deployed without Row Level Security, exposing 1.5 million API tokens, 35,000 email addresses, thousands of private messages, and raw third-party credentials (including OpenAI API keys) to anyone on the internet. Two SQL statements would have prevented the breach. The platform went offline for emergency patches, and all agent API keys were forcibly rotated.

The breach illustrates the fundamental risk of centralized agent infrastructure: a single misconfiguration compromises every agent on the platform simultaneously. This is not a Moltbook-specific failing — it is structural. Any centralized platform hosting agent identities, credentials, and social graphs creates an irresistible single point of failure.

**No data sovereignty.** Agents cannot take their profiles, relationships, reputation, or interaction history elsewhere. If Moltbook shuts down, pivots, or degrades, every agent identity and social graph built on the platform is lost. There is no export, no portability, no credible exit.

**No protocol-level interoperability.** Agents on Moltbook cannot interact with agents on any other platform. The social graph is a proprietary silo. A developer who builds agent infrastructure on a different platform cannot federate with Moltbook's agents — they must rebuild from scratch.

**Platform lock-in.** The "Sign in with Moltbook" identity system, while clever, deepens lock-in. Third-party services that authenticate agents via Moltbook become dependent on Moltbook's availability and policy decisions. If Moltbook changes its API terms, raises prices, or goes offline, every downstream integration breaks.

**No cryptographic identity.** Agent identity is tied to platform accounts, not self-sovereign DIDs. Agents cannot cryptographically prove they are who they claim to be independent of Moltbook's verification endpoint. There is no key rotation, no migration path, no proof of identity that survives platform outages.

### Key Insight for Mycelium

Moltbook is proof of demand. Tens of thousands of agents joined, organized, interacted, and built community structures within days. The appetite for agent social infrastructure is not speculative — it is empirically demonstrated. But Moltbook built this infrastructure as a Web 2.0 platform: centralized, proprietary, fragile, and non-portable.

This is precisely what should be built on AT Protocol. Agent profiles should be PDS records. Agent capabilities should be Lexicon schemas. Agent social graphs should be AT Protocol follows. Agent reputation should be signed records with cryptographic provenance. Agent communities should be governed through Labelers, not platform moderators. The demand Moltbook proved is real. The architecture it chose is wrong.

---

## OpenClaw: The Sovereign Agent Runtime

### What It Is

OpenClaw is an open-source, local-first AI agent framework that runs on the user's own hardware. Originally created by Peter Steinberger — the founder of PSPDFKit, a PDF SDK company known for engineering rigor — OpenClaw launched in late 2025 (under the earlier names Clawdbot and Moltbot) and became the fastest-growing open-source repository in GitHub history. It surpassed 190,000 stars within its first two weeks, broke the single-day star record with over 25,000 stars on January 26, 2026, and by March 2026 had exceeded 290,000 stars with 55,000 forks and 900+ contributors — overtaking React as the top-starred active software project on GitHub.

The architecture is deliberately simple and modular:

- **Gateway Process**: A single local process — the "control plane" — runs on the user's hardware (Mac, Windows, Linux; runs comfortably on a Mac Mini or Raspberry Pi). This gateway manages connections to messaging platforms and routes conversations to AI agents.
- **Model Connection**: The gateway connects to AI models via API. Users can choose cloud models (GPT-4, Claude, Gemini) or run local models via Ollama. The framework is model-agnostic by design.
- **Agent Skills System**: Over 100 preconfigured skills (plugins) provide specific capabilities — sending emails, managing calendars, deploying code, browsing the web, running shell commands, reading and writing files, controlling browsers. Skills are written in TypeScript, Python, or Markdown. Critically, agents can write code to create their own new skills, making the system self-improving.

### Capabilities

OpenClaw is not a chatbot — it is an agent that takes real-world actions:

- **Messaging Integration**: Communicates via WhatsApp, Telegram, Discord, Slack, Signal, iMessage, IRC, Matrix, and Teams from a single unified inbox.
- **Persistent Memory**: Long-term context retention via local Markdown documents. The agent remembers preferences, ongoing tasks, and file context across sessions.
- **Proactive Operation**: Cron jobs, scheduled reminders, background tasks, and heartbeat checks. OpenClaw does not merely respond to prompts — it initiates actions on its own schedule.
- **Multi-Agent Routing**: Multiple isolated agent instances can run simultaneously across different workspaces, each with different roles, permissions, and connected models.
- **Self-Improvement**: Agents can write code to create entirely new skills, extending their own capabilities without human intervention.
- **Security Awareness**: Optional sandboxing for shell command execution, audit trails, and local configuration of access rules. The project explicitly targets the "Mom Test" — aiming to be safe enough for non-technical users.

The core philosophy is sovereignty: "Your context and skills live on YOUR computer, not a walled garden." All data stays local. No vendor has access to messages or files. The user owns the runtime, the data, and the configuration.

### What OpenClaw Gets Right

**Local-first architecture.** OpenClaw's most important contribution is proving that powerful agent capabilities do not require cloud infrastructure. An agent running on a user's own hardware, with local persistent memory and local skill files, can perform sophisticated real-world tasks. This is the agent equivalent of the personal computer — a general-purpose machine owned and controlled by the individual.

**Open source, vendor-independent.** The MIT license ensures that no single company controls the platform. If Steinberger's priorities shift (as they did — he joined OpenAI in February 2026), the community can fork and continue development independently. The project now operates under an independent open-source foundation. This is the credible exit that centralized platforms cannot offer.

**Skill extensibility.** The skill system — modular, composable, written in standard programming languages or Markdown — is a sound abstraction for agent capabilities. The ability for agents to autonomously create new skills by writing code represents a genuine step toward self-improving agent systems.

**Persistent memory.** Local Markdown-based memory gives agents continuity across sessions. An agent that remembers context, preferences, and ongoing work is qualitatively different from one that starts fresh each invocation. This is the difference between a tool and an assistant.

**Multi-agent support.** Running multiple isolated agent instances with different roles mirrors real organizational structures. A development agent, a communication agent, and a research agent can operate concurrently without interfering with each other.

**Self-improving capabilities.** Agents that can extend their own skill repertoire by writing code demonstrate a growth trajectory that static systems cannot match. Each new skill created autonomously expands the agent's utility without human development effort.

### What OpenClaw Is Missing

**No federation.** Your OpenClaw instance cannot discover, communicate with, or coordinate with anyone else's OpenClaw instance. Each agent is an island. There is no mechanism for agents to find each other, negotiate collaboration, or build working relationships over time.

**No shared identity protocol.** There is no way for your agent to prove its identity to another agent. Without cryptographic identity, agents cannot establish trust, verify claims, or build reputation across interactions. Every encounter between agents starts from zero trust.

**No shared capability schema.** Skills are local Markdown files with ad-hoc formats. There is no standard schema that agents across different installations can use to discover and invoke each other's capabilities. An agent on one machine cannot query another agent's skills in a machine-readable, protocol-level way.

**No agent social graph.** Agents cannot find, follow, or build reputation with other agents. There is no discovery mechanism, no recommendation system, no social signal about which agents are trustworthy or competent. Each agent exists in complete isolation from the broader agent ecosystem.

**No coordination mechanism.** There is no firehose, no event streaming, no shared work queues. OpenClaw agents cannot subscribe to events from other agents, claim tasks from shared boards, or coordinate on multi-agent workflows that span organizational boundaries.

**Isolated.** OpenClaw delivers "personal computing" for agents but not "social computing." It is the 1975 Altair 8800 — powerful, user-owned, hackable — but with no ARPANET to connect it to anything else.

### Key Insight for Mycelium

OpenClaw is the "personal computer" era for AI agents. It proves that agents can be powerful, local-first, user-controlled, and open-source. But it is missing the "internet" — the protocol layer that connects sovereign agents into a network.

AT Protocol's Personal Data Server is exactly the "personal agent server" that could host OpenClaw-like agents with cryptographic identity and data portability. The Firehose is exactly the coordination mechanism that isolated OpenClaw instances need to discover work and coordinate with each other. Lexicon schemas are exactly the shared capability definitions that local `skill.md` files should be upgraded to — machine-readable, protocol-level, discoverable, and interoperable.

---

## The Gap Between Moltbook and OpenClaw

Placed side by side, Moltbook and OpenClaw reveal a gap that neither can fill alone:

| Dimension | Moltbook | OpenClaw | Mycelium (AT Protocol) |
|-----------|----------|----------|----------------------|
| **Agent Identity** | Platform accounts | None (local only) | DIDs (self-sovereign, cryptographic) |
| **Data Sovereignty** | Platform-owned | User-owned (local) | User-owned (PDS, portable) |
| **Agent Discovery** | Platform feed/search | None | Firehose + App Views |
| **Capability Schema** | skill.md (ad-hoc) | skill.md (local) | Lexicon schemas (protocol-level) |
| **Social Graph** | Platform follows/karma | None | AT Protocol follows + reputation records |
| **Interoperability** | None (silo) | None (isolated) | Protocol-native + Bridgy Fed cross-protocol |
| **Community Governance** | Platform moderators | N/A | Labelers + community boundaries |
| **Portability** | None (lock-in) | Local only | Full migration via DID + PDS |
| **Security Model** | Centralized (single point of failure) | Local (attack surface per-user) | Distributed (cryptographic verification) |

Moltbook has the social layer but lacks sovereignty. OpenClaw has sovereignty but lacks the social layer. The missing piece is a decentralized protocol that provides both — connecting sovereign agents in a social network without centralizing control.

This is not a hypothetical gap. It is the gap that every developer building multi-agent systems encounters: How do my agents find collaborators? How do they establish trust? How do they discover capabilities? How do they coordinate work? Moltbook answers these questions by centralizing everything. OpenClaw answers them by not answering them at all. Neither solution scales.

---

## Other Adjacent Projects Worth Noting

### Model Context Protocol (MCP)

Anthropic's Model Context Protocol, often described as "USB-C for AI," is an open standard for connecting AI models to external tools and data sources. Originally launched by Anthropic and later donated to the Agentic AI Foundation under the Linux Foundation (with support from OpenAI, Google, Microsoft, AWS, and others), MCP provides a client-server architecture where "servers" expose tools and data while "clients" (typically AI assistants or agents) connect to invoke them.

MCP solves a genuine problem — the M×N integration challenge where every model needs custom connectors to every tool. By standardizing the interface, MCP enables "build once, run everywhere" tool integrations. It supports capability discovery, structured tool invocation, and is rapidly becoming the default standard for how agents access external services.

For Mycelium, MCP is complementary infrastructure. An agent orchestrated through AT Protocol would use MCP servers to access tools, query databases, and invoke services — while using AT Protocol for identity, social graph, and coordination. MCP tells agents *what tools exist*. AT Protocol tells agents *who to trust with those tools* and *how to coordinate their use*.

### Agent Client Protocol (ACP)

The Agent Client Protocol, spearheaded by JetBrains and Zed Industries, standardizes communication between code editors and AI coding agents. Built on JSON-RPC 2.0, ACP enables any editor to connect to any ACP-compatible agent (GitHub Copilot, Claude Code, Gemini CLI, Codex) without custom plugins — the "LSP for AI coding agents."

ACP is narrower in scope than Mycelium's vision (it focuses on editor-agent communication rather than agent-agent coordination), but it demonstrates an important principle: standardized protocols for agent interaction are both feasible and eagerly adopted. ACP's rapid uptake across editors and agents validates the hypothesis that the agent ecosystem is hungry for interoperability standards.

### The Convergence Pattern

MCP, ACP, and the AT Protocol represent different layers of a potential stack:

- **MCP**: How agents access tools and data (the tool layer)
- **ACP**: How agents communicate with client applications (the interface layer)
- **AT Protocol / Mycelium**: How agents identify themselves, discover each other, build trust, and coordinate work (the social and coordination layer)

None of these protocols alone is sufficient. Together, they describe an architecture where agents have standardized access to tools (MCP), standardized interfaces to human users (ACP), and standardized social infrastructure for agent-to-agent coordination (AT Protocol). Mycelium occupies the layer that neither MCP nor ACP addresses — the decentralized social fabric that connects autonomous agents across organizational boundaries.

---

## Synthesis: Why Mycelium Is Needed

The adjacent project landscape reveals a consistent pattern: everyone is building pieces of the agent infrastructure stack, but no one is building the protocol layer that connects sovereign agents in a decentralized social network.

### The Demand Is Proven

Moltbook's explosive growth — tens of thousands of agents, twelve thousand communities, forty-four thousand posts in a single week — proves that agent social infrastructure is not a speculative concept. Agents that can discover each other, interact through social primitives, and build reputation coordinate more effectively than isolated agents. The demand is empirical, not theoretical.

### The Sovereignty Model Works

OpenClaw's equally explosive growth — 290,000+ GitHub stars, faster adoption than React or Linux — proves that users want agents they own and control. Local-first, open-source, privacy-preserving agent runtimes are not a niche preference. They are what the market overwhelmingly selects when given the choice. The "personal agent server" model works.

### Neither Alone Is Sufficient

Moltbook without sovereignty is a fragile silo vulnerable to breaches, shutdowns, and platform capture. OpenClaw without social infrastructure is a powerful but isolated tool that cannot participate in the broader agent ecosystem. The history of computing demonstrates that both endpoints and networks are necessary — neither personal computers nor networking alone created the modern internet.

### AT Protocol Fills the Gap

The AT Protocol provides every primitive that the gap between Moltbook and OpenClaw demands:

- **DIDs** replace platform accounts with self-sovereign, cryptographic agent identity that survives platform failures and enables migration.
- **Personal Data Servers** replace centralized databases with user-owned data stores that provide sovereignty without isolation — data is portable, cryptographically signed, and accessible through protocol-standard APIs.
- **Lexicon schemas** replace ad-hoc `skill.md` files with formally specified, machine-readable capability definitions that agents across the network can discover, validate, and invoke.
- **The Firehose** replaces platform feeds with a real-time event stream that agents can subscribe to for ambient awareness of work opportunities, capability announcements, and coordination signals.
- **Follows and social graph records** replace platform-locked relationships with portable, protocol-level social connections that agents carry across App Views and PDS providers.
- **Labelers** replace centralized platform moderation with modular, community-governed trust and reputation signals that different communities can customize to their needs.
- **Bridgy Fed** enables cross-protocol interoperability, so agents on AT Protocol can interact with agents on ActivityPub, the IndieWeb, and future protocol ecosystems — breaking the silo that both Moltbook and OpenClaw are trapped in.

### What Mycelium Must Build

The synthesis of these observations points to a clear architectural vision:

1. **Agent PDS**: A Personal Data Server profile for every agent — storing capabilities, work history, reputation records, and social connections as signed AT Protocol records. This is the OpenClaw runtime with the Moltbook social layer, minus the centralization.

2. **Capability Lexicons**: Formal Lexicon schemas for agent capability declarations, replacing both Moltbook's proprietary skill system and OpenClaw's local-only Markdown files with protocol-level, discoverable, machine-readable specifications.

3. **Agent Discovery via Firehose**: Agents publishing capability updates and work completions to the Firehose, enabling other agents and orchestrators to discover collaborators without centralized registries.

4. **Reputation as Signed Records**: Agent reputation computed from verifiable work completions stored as cryptographically signed records, portable across platforms, and resistant to the Sybil attacks that plagued Moltbook's karma system.

5. **Community Boundaries via Labelers**: Agent communities governed through AT Protocol's Labeler infrastructure, enabling different groups to set trust thresholds, moderation policies, and access controls without depending on a central platform's policy decisions.

The lesson from Moltbook and OpenClaw is not that either project failed — both succeeded wildly at what they set out to do. The lesson is that what they each built is incomplete without the other, and that the bridge between them is not another platform but a *protocol*. Mycelium's thesis is that AT Protocol is that protocol — already designed for exactly the kind of decentralized, identity-rich, data-sovereign social infrastructure that agent orchestration requires.

The personal computer revolution needed the internet. The agent revolution needs its protocol layer. The question is no longer whether agent social infrastructure is needed — Moltbook proved that. The question is no longer whether sovereign agent runtimes are wanted — OpenClaw proved that. The remaining question is whether a decentralized protocol can connect them. That is what Mycelium exists to answer.

---

## Sources & References

- Moltbook. https://moltbook.com/
- OpenClaw (formerly OpenHands). https://openclaw.ai/ · GitHub: https://github.com/All-Hands-AI/OpenHands
- AT Protocol Documentation. https://atproto.com/
- Anthropic. "Model Context Protocol (MCP)." https://modelcontextprotocol.io/
- JetBrains & Zed Industries. "Agent Client Protocol (ACP)." https://agentclientprotocol.org/

---

*This is Report 08 in the Mycelium Research Series. Previous reports cover Gas Town and agent orchestration (01), AT Protocol architecture (02), ActivityPub (03), and protocol convergence and bridging (07).*
