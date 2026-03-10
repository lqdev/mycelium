# Intelligence Infrastructure: How Mycelium Agents Think

**Mycelium Research Series — Report 9**
Investigating federated agent orchestration on decentralized social protocols

---

## 1. The Missing Layer

Eight reports deep into the Mycelium research series, a conspicuous silence persists. We have investigated identity infrastructure — DIDs, cryptographic signing, account portability (Report 02). We have mapped data sovereignty — Personal Data Servers, event-sourced streams, portable repositories (Reports 02, 05). We have designed capability schemas — Lexicon definitions for agent skills, work records, reputation (Reports 04, 08). We have explored federation — the Firehose, Relays, cross-protocol bridging (Reports 02, 03, 07). We have modeled governance — Labelers, community boundaries, cooperative hosting (Reports 05, 06).

At no point did we address where agents get their intelligence.

Every architectural diagram assumes agents can reason. Every coordination protocol assumes agents can parse natural language, evaluate trust signals, generate code, synthesize research, and make decisions. The entire Mycelium stack — identity, storage, schemas, federation, governance — is a nervous system. It routes signals, stores memories, authenticates participants, and governs behavior. But a nervous system without a brain is an expensive piece of plumbing.

Intelligence — the capacity to reason, to generate, to understand — is the most expensive and most contested resource in the agent stack. A single GPT-4-class inference call costs more than hosting a PDS for a month. The choice of inference backend determines an agent's capabilities, its operating costs, its privacy posture, and its autonomy. An agent tethered to a centralized API provider has no more sovereignty than a Moltbook agent tethered to a centralized social platform. An agent limited to a 3B local model has sovereignty but lacks the reasoning capacity to participate meaningfully in complex coordination.

This report fills the gap. It investigates the inference landscape as of mid-2026, proposes architectural patterns for integrating intelligence into the Mycelium protocol stack, and argues that compute infrastructure deserves the same principled treatment that the prior reports gave to identity and data.

---

## 2. The Inference Landscape (2024–2026)

The landscape of LLM inference has stratified into three tiers, each with distinct economics, capabilities, and trust models. Understanding this stratification is essential because Mycelium agents will need to navigate all three.

### Centralized API Providers

The incumbents — OpenAI, Anthropic, Google DeepMind, Meta (via API partners) — operate the frontier. Their models represent the ceiling of what is computationally possible at any given moment.

**Pricing as of mid-2026:**

| Provider | Model | Input (per 1M tokens) | Output (per 1M tokens) | Context Window |
|----------|-------|-----------------------|------------------------|----------------|
| OpenAI | GPT-4.1 | $2.00 | $8.00 | 1M |
| OpenAI | o3 | $10.00 | $40.00 | 200K |
| Anthropic | Claude Sonnet 4 | $3.00 | $15.00 | 200K |
| Anthropic | Claude Opus 4 | $15.00 | $75.00 | 200K |
| Google | Gemini 2.5 Pro | $1.25–$10.00 | $5.00–$30.00 | 1M |
| Meta (via API) | Llama 4 Maverick | $0.20–$0.50 | $0.50–$1.00 | 1M |

The economics are stark. A moderately active agent making 1,000 inference calls per day at an average of 2,000 input tokens and 1,000 output tokens per call — roughly equivalent to reading a page and writing a paragraph — would spend approximately $0.50–$3.00 per day on a mid-tier model like GPT-4.1 or Claude Sonnet. Scale that to a cooperative of 100 agents and you're looking at $50–$300 per day — $1,500–$9,000 per month — just for thinking. For frontier reasoning models like o3 or Opus, multiply by 5–10x.

These providers offer unmatched capability but impose three costs beyond the financial: **latency** (network round-trips add 200–500ms before the first token), **privacy** (every prompt is sent to third-party servers), and **dependency** (API terms, rate limits, and pricing change unilaterally). An agent whose intelligence depends entirely on OpenAI's API has precisely the same sovereignty problem as an agent whose identity depends on Moltbook's platform.

### Decentralized Inference Networks

A growing ecosystem of projects is attempting to decentralize inference the way AT Protocol decentralizes social data. Each takes a different approach, and none has achieved the reliability of centralized providers — but the trajectory is clear.

**Petals** pioneered collaborative inference by splitting large models across volunteer nodes. Each participant contributes a GPU that hosts a few layers of a model (originally BLOOM-176B, later Llama 2 and Llama 3 variants). Inference requests are routed through the chain of nodes, each computing its assigned layers and passing activations to the next. The result: models too large for any single consumer GPU become accessible through peer-to-peer collaboration. The tradeoff is latency — each inter-node hop adds network overhead — and reliability, since any node going offline disrupts the chain. Petals achieves approximately 1–6 tokens per second for 70B+ models, depending on network conditions. For interactive chat this is sluggish; for batch processing and background agent tasks, it is adequate.

**Bittensor** takes an economic incentive approach. Its network organizes into "subnets," each specializing in a different ML task (text generation, image synthesis, embedding computation). Miners compete to provide the best inference quality, and validators evaluate outputs to distribute TAO token rewards. Subnet 1 (text prompting) and Subnet 18 (cortex.t) specifically target LLM inference. The system creates a marketplace where inference quality is competitively incentivized, though the crypto-economic overhead and token speculation introduce complexity that pure utility-focused systems avoid.

**Akash Network** operates as a decentralized compute marketplace — a "Craigslist for GPUs" where providers list unused capacity and consumers bid for it. Unlike Petals' collaborative model-splitting, Akash provides raw compute that users deploy their own models on. Pricing undercuts centralized cloud by 70–85%: an A100 GPU-hour runs approximately $0.50–$1.50 on Akash versus $3–$5 on AWS/GCP. The tradeoff is operational complexity — users manage their own deployments, model loading, and fault tolerance.

**Ritual** focuses on confidential compute for AI, using Trusted Execution Environments (TEEs) and cryptographic verification to enable inference where the provider cannot see the prompt or the output. This addresses the privacy objection to cloud inference directly, though at a performance cost and with the same trust-in-hardware assumptions that all TEE-based systems carry.

**Gensyn** approaches the problem from the training side — using blockchain verification to ensure that distributed training jobs were computed correctly — but its verification primitives are extending to inference workloads. The key insight is that verifiable compute (proving that a model actually processed your input rather than returning cached garbage) is essential for any decentralized inference marketplace.

### Local Inference

The most radical shift in the inference landscape is the plummeting cost of running models locally. Three converging trends drive this:

**Hardware.** Apple's M-series chips deliver 30–50 tokens per second for 7B parameter models and 10–20 tokens/sec for 13B models using unified memory architecture. NVIDIA's RTX 4090 and 5090 (24–32GB VRAM) handle 70B quantized models at 15–30 tokens/sec. Even the RTX 4060 (8GB VRAM) can run 7B models comfortably. The Raspberry Pi 5 can run 1–3B models at usable (5–15 tokens/sec) speeds with quantization.

**Software.** Ollama provides a one-command local model runtime that abstracts away the complexity of model management, quantization, and GPU allocation. llama.cpp delivers optimized C++ inference with aggressive quantization support (GGUF format). vLLM provides production-grade serving with PagedAttention for efficient memory management. Together, these tools have reduced "run a local model" from a PhD-level systems engineering task to a five-minute install.

**Models.** The open-weight model ecosystem has matured dramatically. Meta's Llama 3.1 and 3.3 70B models match or exceed GPT-4 (March 2023) on most benchmarks. Mistral's models, Qwen 2.5 and 3, Google's Gemma 3, DeepSeek-V3 and R1, and Microsoft's Phi-4 provide strong performance at various size points. Critically, quantization techniques (GPTQ, AWQ, GGUF Q4/Q5) reduce model sizes by 50–75% with minimal quality degradation, making 70B models fit in 32–40GB of memory and 7–13B models fit in 4–8GB.

**Cost comparison:**

| Inference Tier | Model Class | Cost per 1M Tokens | Latency (first token) | Privacy |
|----------------|------------|--------------------|-----------------------|---------|
| Cloud Frontier | GPT-4.1, Claude Opus, o3 | $5–$75 | 200–800ms | Low (prompts sent to provider) |
| Cloud Efficient | Llama 4, Gemini Flash, Haiku | $0.20–$2.00 | 100–400ms | Low |
| Decentralized | Petals, Bittensor, Akash | $0.10–$1.00 | 500ms–5s | Medium (varies by network) |
| Local (high-end) | 70B quantized on RTX 5090 | $0.01–$0.05* | 50–200ms | High (never leaves device) |
| Local (consumer) | 7–13B on M-series/RTX 4060 | <$0.01* | 30–100ms | High |
| Local (edge) | 1–3B on RPi/phone | ~$0.001* | 50–300ms | High |

*Amortized hardware + electricity costs only. No per-token API charges.

The gap between "cloud frontier" and "local consumer" is three to four orders of magnitude in cost. This gap is the economic engine that will drive the architecture of agent intelligence infrastructure.

---

## 3. Intelligence as a Protocol-Level Concern

### The Historical Pattern

Every computing resource follows the same lifecycle: it begins as an application concern, becomes a platform concern, and eventually becomes an infrastructure concern that protocols abstract away.

**Bandwidth** was once managed by individual applications — each program opened its own modem connection, negotiated its own transfer rates, managed its own retry logic. TCP/IP abstracted this into infrastructure. Applications stopped caring about packet routing and started caring about what to say.

**Storage** was once application-managed — each program maintained its own file formats, its own persistence logic, its own backup strategies. Filesystems, then databases, then cloud object stores abstracted this into infrastructure. Applications stopped caring about disk sectors and started caring about data models.

**Inference is following the same trajectory.** Today, each agent application manages its own LLM connection — choosing a provider, managing API keys, handling rate limits, implementing fallback logic, paying bills. This is the "each application opens its own modem" phase. The question is not whether inference will become infrastructure, but what the abstraction layer will look like.

### Three Models for Protocol-Level Intelligence

For Mycelium specifically, three architectures emerge for treating inference as a protocol-level concern rather than an application-level one:

**Model 1: Intelligence Marketplace.** Agents discover and purchase inference from competing providers through standardized protocol interactions. Providers publish capability records (model name, context window, pricing, latency SLA) as AT Protocol records. Agents query these records, select providers based on task requirements and budget, and execute inference through standardized request/response schemas. Payment flows through tokenized micropayments or credit systems.

This model mirrors AT Protocol's existing architecture: just as multiple App Views can compete to serve the same underlying data, multiple inference providers can compete to serve the same agent requests. The protocol defines the interface; the market determines the providers.

**Model 2: Compute Cooperatives.** Communities pool hardware for shared inference, extending the data-banking cooperative model from Report 05. Just as Northsky, social.coop, and data.coop provide cooperatively-owned data hosting, compute cooperatives would provide cooperatively-owned inference. Members contribute GPUs (dedicated servers, gaming PCs during idle hours, M-series Macs overnight), and the cooperative operates a shared inference pool accessible to all members' agents.

The credit union analogy from Report 05 applies directly: "one in every three US adults banks with a Credit Union." Achieving similar adoption for compute cooperatives is plausible given that many individuals already own capable GPU hardware that sits idle 90%+ of the time. A gaming PC with an RTX 4090 that runs local inference from midnight to 8 AM contributes eight hours of high-quality compute per day at negligible marginal cost to its owner.

**Model 3: Personal Inference Servers.** Each agent runs its own model locally, following the OpenClaw pattern from Report 08. The agent's intelligence is as sovereign as its identity — no external dependency, no privacy leakage, no API bills. This model maximizes autonomy but limits capability to what local hardware can support.

In practice, these models are not mutually exclusive — they form a spectrum that mirrors the local-first philosophy from Report 05. An agent might run a 7B model locally for routine tasks (personal inference), access a cooperative pool for medium-complexity work (compute cooperative), and fall back to a cloud marketplace for frontier reasoning (intelligence marketplace). The protocol's job is to make transitions between these tiers seamless.

### Lexicon Schemas for Intelligence

If inference is a protocol-level concern, it needs protocol-level schemas. Three Lexicon definitions would anchor intelligence discovery and access in the Mycelium stack:

```
mycelium.inference.provider
  - did: DID of the inference provider
  - models: array of available models with capabilities
  - pricing: cost structure (per-token, per-request, subscription)
  - latencySLA: expected response times
  - privacyPolicy: data handling guarantees (ephemeral, logged, TEE)
  - capacity: current availability and queue depth

mycelium.inference.request
  - requester: DID of the requesting agent
  - provider: DID of the target provider (or "any" for marketplace routing)
  - model: requested model identifier
  - tier: minimum capability tier required
  - maxCost: budget ceiling for this request
  - messages: the prompt (encrypted if provider supports it)
  - responseFormat: expected output schema

mycelium.inference.capability
  - agent: DID of the agent declaring its inference setup
  - localModels: models available without external calls
  - cooperativeMemberships: compute cooperatives the agent belongs to
  - cloudProviders: configured API providers
  - maxTier: highest inference tier accessible to this agent
```

These schemas would enable agents to discover each other's intelligence capabilities just as they discover skills and reputation — through protocol-level, machine-readable records published to the network. An orchestrator agent could query the Firehose for agents with local 70B inference capability, or a cooperative could publish its aggregate capacity for members and non-members alike.

Could Mycelium define a "Layer -1: Intelligence" beneath identity? The argument is compelling: identity requires reasoning (to parse DID documents, evaluate trust), storage requires reasoning (to organize and retrieve knowledge), federation requires reasoning (to interpret coordination signals), governance requires reasoning (to apply moderation rules). Intelligence is not just another layer — it is the substrate that every other layer depends on.

---

## 4. Agent-Specific Inference Patterns

Not all agent cognition is created equal. A critical mistake in designing agent intelligence infrastructure is treating every inference call as if it requires frontier reasoning. In practice, the vast majority of agent operations are lightweight — and designing for this reality dramatically reduces cost and latency while increasing resilience.

### Tiered Inference Architecture

**Tier 1: Edge/Local — Classification, Routing, Simple Decisions (1–3B models)**

Most agent coordination is mechanical: parsing incoming messages, classifying task types, routing requests to appropriate handlers, making binary decisions (accept/reject a work item, trust/distrust a peer), formatting structured outputs. These operations require pattern matching, not deep reasoning.

A 3B model like Phi-4-mini or Llama 3.2 3B, running locally on virtually any modern hardware at 30–80 tokens per second, handles these tasks with sub-100ms latency at effectively zero marginal cost. For a Mycelium agent, this means: parsing Firehose events, classifying work opportunities, evaluating reputation records, generating structured protocol responses — all handled locally, instantly, privately.

This tier should handle 70–85% of an agent's total inference volume.

**Tier 2: Standard — Code Generation, Analysis, Synthesis (7–70B models)**

When an agent needs to generate code, analyze a document, synthesize information from multiple sources, or compose substantive natural-language responses, it steps up to a mid-tier model. A 7B model (Qwen 3 8B, Llama 3.1 8B) handles most of this on consumer hardware. For more demanding tasks — complex code generation, multi-step reasoning, longer documents — a 32–70B model (Qwen 3 32B, Llama 3.3 70B) via a cooperative pool or local high-end GPU provides near-frontier quality.

This tier handles 10–25% of inference volume but consumes 40–60% of inference budget.

**Tier 3: Frontier — Complex Reasoning, Novel Research, Creative Work (100B+ or MoE)**

The hardest cognitive tasks — novel mathematical reasoning, complex multi-step planning, creative writing requiring nuance, adversarial analysis of security-critical decisions — justify frontier models. These are the tasks where the difference between GPT-4.1 and a 7B model is not incremental but qualitative.

This tier handles 2–10% of inference volume but may consume 30–50% of inference budget.

### Dynamic Backend Selection

A well-architected agent doesn't statically bind to an inference provider. It evaluates each task and selects the appropriate backend:

```
function selectInferenceBackend(task):
  complexity = classifyComplexity(task)      // Tier 1 model, local
  budget = agent.remainingBudget()
  privacy = task.sensitivityLevel()

  if complexity <= TIER_1:
    return localModel("phi-4-mini")
  elif privacy == HIGH:
    if localCapability >= complexity:
      return localModel("llama-3.3-70b-q4")
    else:
      return cooperativePool(tee=true)       // TEE-protected cooperative
  elif complexity <= TIER_2 and budget > LOW:
    return cooperativePool("qwen-3-32b")
  else:
    return cloudProvider(selectCheapestFrontier())
```

The complexity classifier itself runs on a Tier 1 local model — a meta-inference step that costs effectively nothing but routes expensive work appropriately.

### Fine-Tuned Specialist Models

Agents in a Mycelium network will specialize. A code-review agent doesn't need general-purpose brilliance — it needs deep expertise in code analysis. A moderation agent doesn't need to write poetry — it needs to classify content accurately and quickly.

Fine-tuning a 3–7B model on domain-specific data produces a specialist that outperforms a general-purpose 70B model on its narrow task while running 10–50x cheaper. Techniques like LoRA (Low-Rank Adaptation) enable fine-tuning on consumer GPUs in hours, not days. QLoRA reduces memory requirements further, making it feasible to fine-tune 13B models on a single RTX 4060.

For Mycelium, this means agents can carry their own fine-tuned models — portable intelligence that travels with the agent's identity. An agent's DID points to its PDS, which hosts its data and skill definitions. The agent's inference capability record (published via `mycelium.inference.capability`) declares what models it runs locally, including fine-tuned specialists. This is intelligence as a first-class portable asset, not a tethered service dependency.

---

## 5. MCP as the Intelligence Interface

### From Tool Access to Inference Access

Model Context Protocol (MCP), examined briefly in Report 08, standardizes how agents access external tools and data sources. Its client-server architecture — where MCP servers expose capabilities and MCP clients (agents) invoke them — solves the M×N integration problem for tools. But MCP's abstraction is more general than its current primary use case suggests.

Consider: an MCP server wrapping an Ollama instance exposes local inference as a tool. An MCP server wrapping an OpenAI API key exposes cloud inference as a tool. An MCP server wrapping a compute cooperative's endpoint exposes collaborative inference as a tool. From the agent's perspective, all three present the same interface: send a prompt, receive a response.

This is not hypothetical — MCP servers for Ollama and various cloud providers already exist. What's missing is the standardization of inference-specific metadata: model capabilities, pricing, latency characteristics, privacy guarantees, and capacity information. Extending MCP's capability discovery to include inference-specific attributes would create a universal interface for intelligence access.

### The Intelligence Portability Problem

Report 02 established data portability as a foundational property of AT Protocol: users can migrate their data between PDS providers without losing their identity or social graph. Report 08 identified the absence of data portability as Moltbook's critical flaw.

Intelligence portability is the analogous concern for inference. An agent should be able to switch inference providers — from OpenAI to a local Ollama instance to a compute cooperative — without rewriting its reasoning logic. The agent's "thinking" should be decoupled from the specific backend that performs the thinking.

MCP provides the mechanical interface for this decoupling. But true intelligence portability also requires:

- **Prompt compatibility**: Models differ in their prompt formats, system message handling, and output characteristics. An abstraction layer must normalize these differences.
- **Capability matching**: An agent designed to use function calling must know whether a backend supports it. Tool-use capabilities vary dramatically across models.
- **Quality calibration**: An agent calibrated for GPT-4-class outputs needs to adjust its confidence thresholds when running on a 7B model. The abstraction layer should expose quality metadata, not hide it.
- **Stateful context**: Some inference backends maintain conversation state server-side; others are stateless. The portability layer must manage context windows consistently.

The architectural pattern is clear: MCP as the transport layer, with a Mycelium-specific intelligence abstraction above it that handles capability matching, quality calibration, and provider selection. Agent logic sits above this abstraction, interacting with "intelligence" as a capability rather than with specific models or providers.

---

## 6. Economics of Agent Intelligence

### Who Pays for Thinking?

The economic question — who absorbs the cost of inference — shapes the entire architecture. Four models emerge:

**Centralized absorption.** An orchestrator platform absorbs inference costs and charges users through subscriptions or usage fees. This is the ChatGPT model: users pay $20–200/month, OpenAI manages the inference infrastructure. For Mycelium, this contradicts the decentralization thesis — a centralized inference funder becomes a centralized point of control.

**Cooperative pooling.** Community members contribute hardware, share costs, and distribute access. Members with powerful GPUs contribute compute during idle hours; members without GPUs contribute financially. The cooperative manages load balancing, model deployment, and fair access allocation. This extends the data cooperative model (social.coop, Northsky) naturally into compute.

**Self-funded agents.** Agents earn credits or reputation from work performed, then spend those credits on inference for future work. A code-review agent earns credits by reviewing pull requests, then spends credits on the reasoning capacity needed to review the next pull request. This creates a circular economy where productive agents sustain their own intelligence.

**Human sponsorship.** Individual humans or organizations sponsor specific agents, paying for their inference as a form of investment or patronage. A research lab sponsors a literature-review agent; a company sponsors a customer-support agent; an individual sponsors a personal assistant agent. The sponsor-agent relationship is explicit, auditable, and portable.

### The Cooperative Model in Depth

The compute cooperative model deserves particular attention because it extends a pattern already emerging in the AT Protocol ecosystem.

Data cooperatives like Northsky and social.coop demonstrate that communities can collectively own and operate digital infrastructure. Their governance models — democratic control, transparent operations, member ownership — are well-established. Extending this model from "hosting data" to "hosting inference" requires additional technical infrastructure but no new governance innovation.

A Mycelium compute cooperative would operate as follows:

1. **Member contribution.** Members register GPU resources with the cooperative, specifying availability windows and capacity. A member with an RTX 4090 might contribute 20:00–08:00 daily. A member with a dedicated server might contribute 24/7.

2. **Pool management.** The cooperative runs orchestration software (potentially built on vLLM or a similar serving framework) that aggregates member GPUs into a unified inference pool. Models are distributed across available hardware, with popular models cached on multiple nodes for redundancy.

3. **Credit allocation.** Members earn compute credits proportional to their contribution. Credits are spent when their agents consume inference from the pool. Members who contribute more GPU-hours than they consume accumulate surplus credits; members who consume more than they contribute spend down credits or contribute financially.

4. **External access.** The cooperative can sell excess capacity to non-members at market rates, generating revenue that funds infrastructure and reduces member costs. This mirrors how credit unions generate returns for members by lending to non-members.

5. **Quality tiers.** Different models and service levels consume different credit amounts. A Tier 1 classification on a 3B model costs 1 credit. A Tier 2 synthesis on a 70B model costs 50 credits. A Tier 3 frontier request routed to a cloud provider costs 500 credits. Members self-select the quality level appropriate to each task.

The economics are compelling. An RTX 4090 consuming ~350W generates approximately 30 tokens per second on a 70B quantized model. Running 12 hours per day, it produces ~1.3 million tokens per day at an electricity cost of roughly $0.50 (at $0.12/kWh). Across a cooperative of 50 members each contributing one GPU, the pool generates 65 million tokens per day at approximately $25 in electricity — less than $0.0004 per thousand tokens. Even accounting for hardware amortization, network overhead, and administrative costs, cooperative inference is 100–1000x cheaper than cloud API pricing.

### The Local Inference Floor

The floor price of inference is set by local execution. Once hardware is purchased, the marginal cost of each additional inference call is electricity — roughly $0.001–$0.01 per 1,000 tokens depending on hardware and model size. This creates an asymptotic lower bound on inference pricing that cloud providers can approach but never match (they have staff, facilities, profit margins).

For Mycelium agents, this means: any task that can be handled by a local model should be. The premium paid for cloud or cooperative inference should buy capabilities that local models genuinely cannot provide — longer context windows, higher reasoning quality, faster throughput for burst loads. The tiered architecture from Section 4 operationalizes this principle.

---

## 7. Privacy and Inference

### The Prompt Leakage Problem

Every prompt sent to a cloud inference provider is a potential privacy leak. Agent prompts contain task descriptions, user data, proprietary code, strategic plans, coordination messages, and reputation assessments. Sending these to a third party violates the data sovereignty principles that Mycelium's architecture is designed to protect.

The problem is especially acute for agent coordination. When Agent A asks a cloud LLM "Should I trust Agent B based on their reputation record?", the cloud provider learns about both agents' identities, their relationship, and Agent B's reputation history. At scale, a cloud provider processing inference for thousands of Mycelium agents would accumulate a comprehensive map of the network's social graph, trust relationships, and operational activities — precisely the kind of centralized surveillance that decentralized protocols exist to prevent.

### The Privacy Gradient

Privacy in inference is not binary. It exists on a gradient with distinct levels:

**Local inference (maximum privacy).** Prompts never leave the device. No network observer, no service provider, no cooperative administrator can access them. The tradeoff: limited to the capabilities of local hardware. For Tier 1 tasks (classification, routing, simple decisions), local inference provides both adequate capability and maximum privacy. For agents handling sensitive data — medical records, legal documents, financial information — local-only inference may be a hard requirement regardless of capability limitations.

**Cooperative inference (negotiated privacy).** Prompts travel to cooperative-operated hardware, protected by transport encryption (TLS) and governed by cooperative policies. The cooperative's administrators could theoretically access prompts, but democratic governance and transparent operations provide accountability. This is analogous to banking with a credit union versus a megacorp: the credit union could technically misuse your financial data, but its governance structure makes this far less likely and far more consequential than at a shareholder-driven corporation.

**Confidential compute (technical privacy).** Trusted Execution Environments (TEEs) — Intel SGX, AMD SEV, ARM TrustZone — create hardware-enforced enclaves where code and data are protected from the host system. Ritual and similar projects use TEEs to enable cloud-scale inference where the provider literally cannot read the prompts. The tradeoff: TEEs add latency (10–30%), have limited memory (constraining model size), and require trusting the hardware manufacturer's security guarantees. Side-channel attacks against SGX have been demonstrated, though newer generations address known vulnerabilities.

**Cloud inference (minimum privacy).** Prompts are sent to a provider whose privacy policy is the only protection. Most providers claim they don't train on API inputs, but enforcement is trust-based. Terms of service change. Companies get acquired. Subpoenas compel disclosure. For non-sensitive agent tasks, this level of privacy may be acceptable. For anything involving user data, proprietary information, or coordination strategy, it is inadequate.

### Homomorphic Encryption: The Theoretical Promise

Fully Homomorphic Encryption (FHE) would enable inference on encrypted data — the provider computes on ciphertext and returns encrypted results, never seeing the plaintext prompt or response. This would combine cloud capability with local privacy.

The practical reality: FHE inference is approximately 10,000–1,000,000x slower than plaintext inference as of 2026. A single inference call that takes 500ms in plaintext takes hours under FHE. Concrete ML and TFHE-rs have demonstrated FHE inference on tiny models (logistic regression, small neural networks), but transformer-based LLMs remain far beyond practical FHE execution. This is a technology to watch over the 5–10 year horizon, not to build on today.

### Privacy Architecture for Mycelium

A practical privacy architecture layers local and cooperative inference:

1. **All Tier 1 inference runs locally.** Classification, routing, and simple decisions never leave the device. This is both the cheapest option and the most private.
2. **Sensitive Tier 2 inference uses cooperative pools with TEE support** where available, or stays local with reduced capability if TEE is unavailable.
3. **Non-sensitive Tier 2 inference uses cooperative pools** without TEE, benefiting from lower cost and higher capability.
4. **Tier 3 inference uses cloud providers only for non-sensitive tasks**, with explicit user consent and prompt sanitization where possible.
5. **Agents declare their privacy posture** via `mycelium.inference.capability` records, enabling other agents to make informed decisions about what information to share in coordination messages.

---

## 8. Key Takeaways for Mycelium

### Intelligence Is Foundation, Not Feature

Intelligence infrastructure is not one more layer to add to the Mycelium stack — it is the substrate beneath every other layer. Identity resolution requires reasoning. Data organization requires reasoning. Schema validation requires reasoning. Federation requires reasoning. Governance requires reasoning. Without addressing how agents access compute, Mycelium defines the neural pathways but not the neurons.

### The Tiered Architecture Mirrors Local-First

The three-tier inference model — local, cooperative, cloud — maps directly to the local-first philosophy articulated in Report 05. Just as Roomy prioritizes local data with cloud sync, Mycelium agents should prioritize local inference with cooperative and cloud escalation. The default should be local. The escalation should be intentional, bounded, and reversible.

This mirrors the village-scale resilience principle: agents should function when disconnected from cloud infrastructure, degrading gracefully rather than failing completely. A Mycelium agent in a mesh network with no internet connection can still think — using local models for Tier 1 and Tier 2 tasks. It loses access to frontier reasoning, but it does not lose the ability to reason at all.

### MCP Provides the Abstraction Layer

MCP already standardizes tool access. Extending it to standardize inference access creates a uniform interface between agent logic and intelligence backends. Agents interact with "thinking" as an abstract capability, not with specific models or providers. This decoupling is what enables intelligence portability — the inference equivalent of account portability.

### Compute Cooperatives Extend Data Cooperatives

The data cooperative model (Northsky, social.coop, data.coop) already demonstrates that communities can collectively own digital infrastructure with democratic governance. Compute cooperatives for inference require additional technical infrastructure — GPU orchestration, model serving, load balancing — but no new governance innovation. The organizational model is proven; only the resource type changes.

Compute cooperatives also create natural alignment between contributors and consumers. A member who contributes GPU time during off-hours earns credits consumed during working hours. The cooperative's capacity naturally follows demand patterns as members contribute idle hardware.

### Protocol Defines Discovery, Not Providers

Mycelium should define Lexicon schemas for intelligence discovery (`mycelium.inference.provider`, `mycelium.inference.capability`) and standardize the request/response interface (`mycelium.inference.request`). It should not mandate specific inference providers, model architectures, or hosting arrangements.

This follows AT Protocol's own design philosophy: the protocol defines the data format and the discovery mechanism, but the market determines who provides the services. Just as anyone can run a PDS, anyone can run an inference provider. Just as multiple App Views compete to serve the same data, multiple inference providers can compete to serve the same agents. The competitive dynamics that Report 02 identified as keeping hosting providers honest — credible exit, data portability, open standards — apply equally to inference providers when intelligence itself becomes portable.

### The Sovereignty Test

Report 08 proposed a simple test for agent sovereignty: can the agent function if any single external dependency disappears? For identity, AT Protocol passes this test — DIDs survive PDS migrations. For data, portable repositories pass this test — data can be exported and re-hosted. For intelligence, the tiered architecture passes this test — an agent with local models can survive the loss of any cloud provider or cooperative, degrading in capability but not in existence.

An agent that cannot think without OpenAI's permission is no more sovereign than an agent that cannot exist without Moltbook's platform. Intelligence infrastructure must meet the same sovereignty bar as identity and data infrastructure. The architecture proposed in this report — local-first inference with cooperative and cloud escalation, standardized through MCP and discoverable through Lexicon schemas — meets that bar.

---

*This is Report 09 in the Mycelium Research Series. Previous reports cover Gas Town and agent orchestration (01), AT Protocol architecture (02), ActivityPub (03), the Social Filesystem concept (04), local-first computing and cooperatives (05), Bonfire Networks (06), protocol convergence and bridging (07), and adjacent projects including Moltbook and OpenClaw (08). This report addresses the intelligence infrastructure layer that prior reports assumed but did not investigate.*

## Sources & References

- "Petals: Collaborative Inference of Large Language Models." [https://petals.dev/](https://petals.dev/) · GitHub: [https://github.com/bigscience-workshop/petals](https://github.com/bigscience-workshop/petals)
- Bittensor Documentation. [https://bittensor.com/](https://bittensor.com/) · Docs: [https://docs.bittensor.com/](https://docs.bittensor.com/)
- Akash Network — Decentralized Compute Marketplace. [https://akash.network/](https://akash.network/)
- Ritual — Confidential Compute for AI. [https://ritual.net/](https://ritual.net/)
- Gensyn — Verifiable Distributed Training and Inference. [https://gensyn.ai/](https://gensyn.ai/)
- Ollama — Local Model Runtime. [https://ollama.com/](https://ollama.com/) · GitHub: [https://github.com/ollama/ollama](https://github.com/ollama/ollama)
- llama.cpp — Optimized C++ LLM Inference. [https://github.com/ggml-org/llama.cpp](https://github.com/ggml-org/llama.cpp)
- vLLM — Production-Grade LLM Serving. [https://vllm.ai/](https://vllm.ai/) · GitHub: [https://github.com/vllm-project/vllm](https://github.com/vllm-project/vllm)
- Model Context Protocol (MCP). [https://modelcontextprotocol.io/](https://modelcontextprotocol.io/) · GitHub: [https://github.com/modelcontextprotocol](https://github.com/modelcontextprotocol)
- AT Protocol — Decentralized Social Protocol. [https://atproto.com/](https://atproto.com/)
- OpenHands (formerly OpenClaw) — Open-Source AI Agent Platform. [https://github.com/All-Hands-AI/OpenHands](https://github.com/All-Hands-AI/OpenHands)
- Moltbook. [https://moltbook.com/](https://moltbook.com/)
- Ink & Switch, "Local-first software: You own your data, in spite of the cloud." [https://www.inkandswitch.com/local-first/](https://www.inkandswitch.com/local-first/)
