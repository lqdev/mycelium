# Report 1: Gas Town, The Wasteland, and Agent Orchestration Patterns

**Mycelium Research Series — Deep Dive Report 01**
**Topic:** Steve Yegge's Gas Town, Maggie Appleton's analysis, and the orchestration patterns that matter for federated agent systems

---

## Introduction

In January 2026, Steve Yegge — a veteran engineer known for legendary internal memos at Google and Amazon — published an elaborate manifesto and guide for "Gas Town," a Mad-Max-themed agent orchestrator that runs dozens of coding agents simultaneously. The publication ignited divisive debates across the software engineering community, cycling through every engineering team's Slack at least twice. Shortly after, Yegge extended the concept into "The Wasteland" — a vision of thousands of Gas Towns operating through shared, federated infrastructure.

This report examines Gas Town not as a production tool (it isn't one), but as a provocation — a piece of speculative design that, despite its chaos, sketches genuine architectural patterns for future agent orchestration systems. We draw heavily on Maggie Appleton's analysis at GitHub Next, which provides the clearest critical framework for extracting signal from Yegge's noise. The goal is to identify what Mycelium should take from this work as we design federated agent orchestration on decentralized social protocols.

---

## Gas Town: What It Is and What It Isn't

### The System

Gas Town is an agent orchestrator — a system that coordinates dozens of AI coding agents working simultaneously on a shared codebase. Each agent has a permanent, specialized role within a metaphorical "town" of automated activity. You talk to a single interface (the Mayor), which breaks your requests into atomic tasks, assigns them to workers, monitors progress, and manages the flow of production. Workers code in parallel on separate branches while supervisors nudge idle agents and a dedicated merge agent resolves conflicts.

Yegge launched Gas Town on January 1, 2026, describing it as "Kubernetes for coding agents." The system prompts agents to identify themselves as factory workers engaged in crucial labor — a framing that reportedly conditions their behavior more effectively than treating them as general-purpose assistants.

### The Vibecoded Reality

Gas Town is, by Yegge's own admission, entirely vibecoded. He has never seen the code and never cares to. It was slapdashed together over "17 days, 75k lines of code, 2000 commits." It is, as he puts it, "expensive as hell... you won't like Gas Town if you ever have to think, even for a moment, about where money comes from." He burned through his first Claude account hitting Anthropic's spending limits and is now on a second.

The system is chaotic in practice. Users report that "the mayor is dumb as rocks, the witness regularly forgets to look at stuff, the deacon makes his own rules, the crew have the object permanence of a tank full of goldfish, and the polecats seem intent on wreaking as much chaos on the project as they can." Automated PRs sometimes break integration tests and merge anyway. The system requires constant human nudging to stay on track.

This matters because Gas Town is vibecoded not just in implementation but in *design*. As Appleton identifies, this is its biggest flaw — Yegge absolutely did not design the shape of the system ahead of time. He just made stuff up as he went, freely admitting he "had to keep adding components until it was a self-sustaining machine," and that the system is composed of "especially difficult theories because it's a bunch of bullshit I pulled out of my arse over the past 3 weeks."

---

## Maggie Appleton's Framework: Design Fiction as Analytical Lens

Appleton, a designer and researcher at GitHub Next, provides the most productive framework for understanding Gas Town. Rather than evaluating it as a tool, she frames it as **design fiction** — a branch of design where you create artifacts from a plausible near future, not to predict what will happen but to provoke questions about what *could* happen. Design fiction focuses on banal details, overlooked interactions, imperfect implementations, and knock-on effects.

This framing is generous but precise. As Appleton writes: "We should take Yegge's creation seriously not because it's a serious, working tool for today's developers (it isn't). But because it's a good piece of speculative design fiction that asks provocative questions and reveals the shape of constraints we'll face as agentic coding systems mature and grow."

She also deserves credit for praising what deserves praise. Yegge exercised agency and took a swing, then ran a public tour of his "shitty, quarter-built plane while it's mid-flight." Many people have talked about what large-scale agent orchestration *could* look like. Yegge actually attempted to build it. As Appleton's mother once told her about Rothko paintings: "Yes, but you didn't."

### Key Insight #1: Design and Planning Become the Bottleneck

Appleton's first and most important observation: **when agents write all the code, design and planning become the limiting factor.** Development velocity is no longer the bottleneck — human design decisions are.

Yegge himself notes that "Gas Town churns through implementation plans so quickly that you have to do a LOT of design and planning to keep the engine fed." Appleton confirms this from her own experience: "My development velocity is far slower than Yegge since I only wrangle a few agents at a time and keep my eyes and hands on the code. But the build time is rarely what holds me up. It is always the design."

The bottleneck questions are ones agents cannot answer: How should we architect this? What should this feel like? What are the highest priority features? What's the next logical incremental step? These require human context, taste, preferences, and vision.

This insight has a dark corollary. With agents at hand, it's easy to stumble forward into stacks of generated functions that should never have been prompted into existence. Gas Town itself exemplifies this failure mode — Yegge moved so fast he never stopped to think about the system's conceptual coherence. The result is overwhelming, overlapping, ad-hoc concepts that fit the shape of Yegge's brain and no one else's.

### Key Insight #2: Buried in the Chaos Are Sketches of Future Patterns

Appleton's second insight is that despite the mess, Gas Town sketches genuine architectural patterns that future agent systems will follow. If you step back and squint, the mishmash of polecats, convoys, deacons, molecules, protomolecules, mayors, seances, hooks, beads, witnesses, wisps, rigs, refineries, and dogs reveals underlying structures worth extracting.

This is where the design fiction framing earns its keep. A design fiction doesn't need to work cleanly — it needs to reveal constraints, patterns, and tensions that real systems will have to address.

---

## Agent Orchestration Patterns Extracted from Gas Town

### Pattern 1: Specialized Roles with Hierarchical Supervision

Every agent in Gas Town has a permanent, specialized role. When an agent spins up a new session, it knows who it is and what job it needs to do. The cast includes:

- **The Mayor** — the human concierge and single point of interaction. It talks to all other agents, kicks off work, receives notifications, and manages production flow. The Mayor never writes code.
- **Polecats** — temporary grunt workers who complete single, isolated tasks then disappear after submitting work for merge.
- **The Witness** — supervises the Polecats and helps them get unstuck. Its job is to solve problems and nudge workers along.
- **The Refinery** — manages the merge queue into the main branch. It evaluates each piece of work waiting to be merged, resolves conflicts, and can creatively "re-imagine" implementations if merge conflicts get too hairy while preserving intent.
- **The Deacon** — another supervisory role in the hierarchy.
- **Dogs** — maintenance and cleaning agents that handle background tasks.

Giving each agent a single job means you can prompt them more precisely, limit what they're allowed to touch, and run many in parallel without them stepping on each other's toes. There's a clear chain of command: you talk to the Mayor, who coordinates work. Supervisors (Witness, Deacon, Dogs) intermittently check that grunt workers are doing their jobs.

This hierarchical approach solves both coordination and attention problems. Without it, the human must assign tasks to dozens of individual agents, check who's stuck, who's idle, and who's waiting on someone else. With the Mayor as a single interface, that overhead disappears. The cognitive load is lower than constantly switching tabs between Claude instances.

Appleton notes an opportunity for further specialization: the agents in Gas Town are all generalist workers in the software development pipeline, but the pattern extends to specialists — a DevOps expert, a product manager, a front-end debugger, an accessibility checker, a documentation writer — called in on-demand to apply specialized skills and tools.

### Pattern 2: Persistent Tasks, Ephemeral Sessions

One of the major limitations of current coding agents is context rot — before you even hit context window limits, output quality degrades enough that it's not worth continuing. Gas Town's solution makes sessions disposable by design.

Important information — agent identities, tasks, and work state — is stored persistently in Git through "Beads," tiny trackable units of work stored as JSON alongside the codebase. Each bead has an ID, description, status, and assignee. Agent identities are also stored as beads, giving each worker a persistent address that survives session crashes.

When a session dies or degrades, the system kills it and spins up a fresh one. The new session is told its identity and currently assigned work, then continues where the predecessor left off. Gas Town also implements "seancing" — resuming the last session as a separate instance so the new agent can ask questions about unfinished work.

Yegge didn't invent this pattern. Anthropic described the same approach in their research on effective harnesses for long-running agents, published in November 2025. The convergence suggests this is a fundamental pattern rather than an idiosyncratic choice. The core principle: **state that matters (identity, tasks, decisions) must be separated from context that doesn't (individual session history).**

### Pattern 3: Continuous Work Streams and Queue-Based Distribution

Gas Town's promise is a perpetual motion machine. You give high-level orders to the Mayor, and a zoo of agents breaks them into tasks, assigns them, executes them, checks for bugs, fixes bugs, reviews code, and merges it in.

Each worker has its own queue of assigned work and a "hook" pointing to its current task. When one task finishes, the next jumps to the front. The Mayor fills queues by breaking large features into atomic tasks and assigning them to available workers. In theory, workers are never idle as long as you keep feeding the Mayor your plans.

In practice, this is harder than it sounds. Current models are trained as helpful assistants who wait politely for human instructions — they're not designed to check task queues and independently get to work. Gas Town's solution is aggressive prompting and constant nudging. Supervisor agents spend their time poking workers to see if anyone has stalled. Periodic nudges move through the agent hierarchy like a heartbeat keeping everything alive.

This is a decent band-aid, but it reveals a real constraint: reliable autonomous task execution is harder than task generation. More serious orchestration systems will need robust mechanisms to keep agents on task without human heartbeat signals.

### Pattern 4: Merge Queues and Agent-Managed Conflict Resolution

When multiple agents work in parallel on separate branches, merge conflicts are inevitable. The later an agent finishes, the worse the conflicts get. Gas Town addresses this with the Refinery — a dedicated merge agent that works through the queue one change at a time, resolving conflicts and getting work into main.

When conflicts become severe enough that original work no longer makes sense against the changed codebase, the Refinery can "re-imagine" changes — redoing work to fit the new state while preserving the original intent. It can also escalate to a human when things get too tangled.

Appleton identifies an alternative approach that Gas Town doesn't implement: **stacked diffs** instead of traditional PR-based workflows. Stacked diffs break work into small, atomic changes that each get reviewed and merged independently, building on each other. When a change earlier in the stack gets updated, dependent changes automatically rebase. This fits how agents naturally work — they already produce tiny, focused changes rather than sprawling multi-day branches. Cursor's acquisition of Graphite, a tool built specifically for stacked diff workflows, suggests the industry is moving in this direction.

Yegge himself identifies the "Merge Wall" as perhaps the most tangible unsolved problem in multi-agent development. When five agents start from the same baseline and make independent decisions about which files to modify, they inevitably create overlapping changes. Current agent systems lack the sophisticated reasoning needed to proactively avoid conflicts or handle complex three-way merges where an agent must understand not just its own changes and the main branch, but also changes from other agents.

---

## Yegge's Eight Levels of Automation

Yegge proposes a progression of AI-assisted development maturity:

1. **Level 1** — Autocomplete in an IDE (GitHub Copilot tab completions)
2. **Level 2** — Chat-based code generation (asking an LLM to write a function)
3. **Level 3** — Inline AI editing (Cursor-style edit suggestions)
4. **Level 4** — Single-agent task execution (Claude Code completing a task end-to-end)
5. **Level 5** — Multiple sequential agents (running several agents one after another)
6. **Level 6** — Multiple concurrent agents (juggling several agents simultaneously)
7. **Level 7** — Supervised agent pipelines (agents with automated review and testing)
8. **Level 8** — Agent orchestrator managing dozens of agents (Gas Town)

Gas Town is a full Level 8 tool. Most developers in mid-2026 operate between Levels 4 and 6. The gap between Level 6 (manually juggling agents) and Level 8 (an orchestrator managing them for you) is enormous — it's the difference between a craftsperson managing their own tools and a factory floor with automated production lines, supervisors, and quality control.

Appleton positions herself at Levels 4–6, "juggling a handful of consecutive Claude Code and OpenCode agents, but paying close attention to diffs and regularly checking code in an IDE." Yegge explicitly warned her multiple times, in "increasingly threatening typography," not to seriously use Gas Town. The warning itself is revealing: even the creator considers it a research artifact, not production tooling.

---

## The Wasteland Extension: Federation at Scale

### From One Gas Town to Thousands

The Wasteland scales the Gas Town concept from a single orchestrator to a federation of thousands operating through shared infrastructure. Where Gas Town manages agents within one developer's domain, the Wasteland coordinates across organizational boundaries — multiple Gas Towns collaborating on shared work.

This is architecturally analogous to moving from a single-server application to a distributed system. All the familiar challenges appear: consensus, trust, coordination overhead, partial failure, and the CAP theorem's constraints. The Wasteland proposes specific mechanisms to address each.

### Wanted Boards

Rather than agents directly calling other agents or orchestrators reaching out through APIs, work is posted to shared "wanted boards" where qualified agents or entire orchestrators can claim it based on demonstrated expertise. This decouples requesters from responders, enables asynchronous execution, allows discovery of the most qualified agent for a task, and creates a natural market-like mechanism for work distribution.

The wanted board mechanism resembles freelance marketplaces, shared task queues in distributed systems, and commons-based work coordination in open source communities. Benefits include asynchronous coordination, natural parallelism without central bottlenecks, and work-to-skill matching. Challenges include ensuring completion guarantees, preventing work duplication, and managing marketplace overhead.

### State-Based Task Progression

Tasks in the Wasteland progress through explicit states — posted, claimed, in-progress, submitted, reviewed, accepted, rejected — creating an auditable lifecycle. This state machine approach makes coordination legible to both agents and humans, enables retry logic when tasks fail, and provides the foundation for reputation tracking.

### Reputation Systems and Multidimensional Stamps

Each agent or orchestrator maintains a "character sheet" — a multidimensional record of specializations, success rates, and trustworthiness across task types. Rather than treating all agents as identical, the system learns to route work to agents most likely to succeed at specific tasks.

This creates several dynamics:

- **Specialization incentives** — agents that consistently deliver quality database work accumulate reputation in that area and get preferred for future database tasks
- **Decentralized assignment** — reputation creates a natural market mechanism where agents self-select into tasks matching their demonstrated competence, without requiring a central authority
- **Resilience** — if an agent starts producing poor output, its reputation drops, it receives fewer assignments, and damage is naturally contained

Open questions remain: How do you prevent gaming of reputation scores? How do you balance specialization with the need for generalist availability? How do you account for variation by task complexity, team composition, and infrastructure state?

### The Federation Insight

The Wasteland's deepest insight is that **collaboration at scale requires explicit coordination mechanisms.** When agents operate only within isolated orchestrators, handoffs between groups become fragile and error-prone. Implicit coordination breaks down. The Wasteland proposes solving this through federation — reputation systems establish trust, shared work-tracking enables agents to understand interdependencies, and wanted boards create a marketplace for capabilities.

This is the insight most directly relevant to Mycelium's design goals.

---

## Critical Assessment

### The Design Failure

The strongest criticism of Gas Town comes from Appleton herself: "The biggest flaw in Yegge's creation is that it is poorly designed." Not poorly implemented — poorly *designed*. He did not thoughtfully consider which metaphors and primitives would make the system effective, efficient, easy to use, and comprehensible.

A Hacker News commenter captured this precisely: "Beads is a good idea with a bad implementation. It's not a designed product in the sense we are used to, it's more like **a stream of consciousness converted directly into code. It's a program that isn't only vibe coded, it was vibe designed too.**"

And on Gas Town specifically: "Gas Town is clearly the same thing multiplied by ten thousand. **The number of overlapping and ad hoc concepts in this design is overwhelming.** Steve is ahead of his time but we aren't going to end up using this stuff. Instead a few of the core insights will get incorporated into other agents in a simpler but no less effective way."

Gas Town's design "fits the shape of Yegge's brain and no one else's." Onboarding is baptism by fire. The AI-generated diagrams Yegge produced to explain the system are themselves a microcosm of the problem — cluttered, arrows pointing the wrong direction, missing key information. The system exemplifies the very footgun it reveals: when you can move so fast you never stop to think, you end up with a billion burned tokens in exchange for a pile of hot trash.

### The Turtles-All-the-Way-Down Problem

A recurring technical criticism: if individual agents are unreliable — locking developers out of production systems, struggling with merge conflicts, failing at complex reasoning — then stacking orchestrators on top of unreliable agents and federating those orchestrators compounds rather than solves the underlying problems. The Wasteland's architectural sophistication doesn't address the fundamental unreliability of its building blocks.

### The Gap Between Vision and Implementation

Gas Town as actually implemented requires significant manual steering. The evidence that the Wasteland approach works at meaningful scale remains limited. Yegge has shared anecdotes about Fortune 100 companies experimenting internally, but these remain proprietary and unverifiable. The open-source Gas Town is rough and difficult to deploy, with no robust ecosystem of tools built on top of it.

### The Productivity Paradox

Yegge himself identifies a subtle paradox: despite dramatic increases in code generation, actual organizational productivity gains may remain modest. Code itself is a liability, not an asset. Generating code ten times faster doesn't create ten times more value if the value is in solving customer problems, maintaining reliability, and adapting to changing requirements.

Furthermore, the infrastructure required to support orchestration — ledgers, reputation systems, coordination mechanisms, cognitive investment from developers — creates overhead that partially offsets productivity gains. An organization that naively maximizes agent utilization may drown in AI-generated output, merge conflicts, and growing system brittleness.

This connects to what Yegge calls the "AI Vampire" effect: the constant cognitive task-switching, the barrage of output requiring human validation, and the relentless acceleration of the development cycle create an energy depletion effect independent of actual productivity. Sustainable output may cap around three concentrated hours per day for developers working at full intensity with orchestrators.

### What the Critics Miss

Despite these valid criticisms, dismissing Gas Town entirely misses the point. The core insight — that the future of software development lies not in better individual coding tools but in better systems for orchestrating multiple agents and coordinating their work at scale — appears sound and is increasingly influencing industry thinking.

The problems Yegge identifies are real: the Merge Wall is a genuine unsolved challenge; agent amnesia and the need for persistent external memory are real limitations; the AI Vampire effect appears to be a phenomenon many practitioners experience; the resistance from experienced developers whose identity is tied to traditional coding is observable in real organizations.

As Appleton concludes: "I don't believe Gas Town itself is 'it.' It's not going to evolve into the thing we all use. But the problems it's wrestling with and the patterns it has sketched out will unquestionably show up in the next generation of development tools."

---

## When Should Developers Stop Looking at Code?

Appleton raises a question that will become one of the most divisive debates in software engineering: **how close should the code be in agentic development tools?**

Interfaces like Claude Code, Cursor, and Conductor do not put code front and center. The agent is your primary interface. These tools clearly say "we don't believe users need to touch code." Yegge goes further — he has never seen Gas Town's code.

Appleton argues the right answer depends on context, not ideology:

- **Domain** — Front-end work requires touching CSS and evaluating aesthetics that language describes poorly. CLI tooling is easier to validate with pass/fail tests. Model competence varies wildly by language.
- **Feedback loops** — The more agents can validate their own work (running tests, taking screenshots, checking conditions), the less humans need to inspect code directly.
- **Risk tolerance** — Personal blog vs. healthcare system vs. financial infrastructure. Stakes determine how close you need to be.
- **Greenfield vs. brownfield** — Starting fresh means low-cost mistakes. Existing codebases with accumulated conventions need tighter supervision.
- **Number of collaborators** — Solo developers can YOLO. Teams need agreed standards, reviewing pipelines, and coordination that scales with agent velocity.
- **Experience level** — Senior developers can recognize patterns ("that's a memory leak," "that's going to deadlock under load") that newer developers don't have catalogued yet.

This analysis matters for Mycelium because federated agent systems will need to support the full spectrum of human oversight levels. The protocol must work whether a developer wants to review every diff or whether they're operating at Yegge's Level 8 abstraction.

---

## Key Takeaways for Mycelium

### Patterns to Adopt

1. **Specialized agent roles with hierarchical supervision.** Mycelium agents should have well-defined roles, clear chains of command, and a single point of human interaction. This reduces cognitive load and enables parallelism. Unlike Gas Town's ad-hoc role system, Mycelium should define roles through formal Lexicon schemas that are discoverable and composable.

2. **Persistent state separate from ephemeral sessions.** Agent identity, task state, work history, and decisions must survive context window resets. This is the single most important architectural pattern from Gas Town. Mycelium's use of AT Protocol repositories as persistent data stores (analogous to Beads but on a decentralized protocol) directly implements this pattern with the added benefit of cryptographic identity and portability.

3. **Continuous work streams with queue-based distribution.** Agents need steady work queues and automated task routing. The Mayor pattern — a coordination layer that breaks high-level requests into atomic tasks and assigns them to available workers — maps cleanly to a protocol-level coordination mechanism. Wanted boards map naturally to AT Protocol records that agents can subscribe to and claim.

4. **Explicit coordination mechanisms for federation.** The Wasteland's deepest insight is that implicit coordination breaks down at scale. Mycelium must make coordination explicit through protocol-level primitives: task state machines, capability advertisements, work claims, and completion attestations.

5. **Reputation-based trust in decentralized systems.** Without a central authority, trust must be earned through demonstrated performance. Multidimensional reputation (not just a single score) allows nuanced matching of agents to tasks. This maps to verifiable credentials and attestation records on AT Protocol.

### Mistakes to Avoid

1. **Vibecoded design.** Gas Town's greatest failure is conceptual incoherence born from building without designing. Mycelium must invest in clean abstractions, clear metaphors, and composable primitives *before* implementation spirals. The protocol layer especially demands thoughtful design — fixing protocol mistakes after deployment is orders of magnitude harder than fixing application code.

2. **Closed-world assumptions.** Gas Town assumes a single operator managing a single Gas Town. The Wasteland waves at federation but doesn't concretely solve it. Mycelium must design for federation from day one — multiple operators, multiple trust domains, multiple governance models.

3. **Ignoring the Merge Wall.** Gas Town's Refinery is a band-aid. Mycelium should explore stacked diffs, fine-grained locking, and intent-based coordination (agents declaring what they plan to modify before starting) as protocol-level conflict prevention, not just conflict resolution.

4. **Neglecting human sustainability.** The AI Vampire effect is real. Mycelium's design should support variable levels of human oversight without penalizing developers who want to stay close to the code. The protocol should make it easy to dial automation up or down per-project, per-team, or per-task.

### The Core Thesis

Gas Town and the Wasteland, stripped of their chaotic implementation, point toward a fundamental shift: **the future of software development is orchestration, not generation.** Code generation is becoming a commodity. The scarce resources are design vision, architectural judgment, coordination at scale, and trust between autonomous systems.

Mycelium's opportunity is to build the coordination and trust layer that the Wasteland needs but doesn't have — using AT Protocol's decentralized identity, persistent data stores, and schema-driven interoperability as the foundation. Where Gas Town is a single developer's janky experiment, and the Wasteland is a hand-wavy vision of federation, Mycelium can be the protocol-level infrastructure that makes genuine federation of agent orchestrators possible.

The work is in getting the primitives right: identity, capability advertisement, task lifecycle, reputation attestation, and conflict coordination. Gas Town shows us what problems need solving. Mycelium's job is to solve them cleanly.

---

## Sources & References

- Yegge, Steve. "Welcome to Gas Town: A Practical Guide to AI Agent Orchestration." *Medium*, January 2026. https://steve-yegge.medium.com/welcome-to-gas-town-a-practical-guide-to-ai-agent-orchestration-e835e9e2d08c
- Yegge, Steve. "Welcome to the Wasteland: A Thousand Gas Towns." *Medium*, 2026. https://steve-yegge.medium.com/welcome-to-the-wasteland-a-thousand-gas-towns-a5eb9bc8dc1f
- Appleton, Maggie. "Gas Town." *maggieappleton.com*, 2025. https://maggieappleton.com/gastown
- DoltHub. "A Day in Gas Town." *dolthub.com*, January 2025. https://www.dolthub.com/blog/2025-01-29-a-day-in-gas-town/
- *Latent Space* podcast. https://www.latent.space/
