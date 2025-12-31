# Agentic Workflow Orchestration Stack

But agentic chaos isn’t inevitable.  
Not if you have a systems view.

Here’s a clean, **9-layer model** to design real multi-agent workflows — **and actually ship them**:

---

![Agentic Workflow Stack](images/agentic-workflow-stack-1.png)

## Level 0: Deployment  
**Where the agents actually live**

This is the substrate: latency, cost, throughput, regionality, GPU availability, cold starts, and runtime constraints. If this layer is shaky, every “agent bug” above it becomes impossible to diagnose.

**Design questions**
- Where does inference run: hosted API, self-hosted, hybrid?
- What’s the scaling unit: request, session, workflow, tenant?
- What’s the failure posture: retry, degrade, queue, fall back?

**Common failure modes**
- Hidden rate limits → cascading timeouts  
- “Works locally” → dies under concurrency  
- Cold-start penalties → agents “think” slower than your users tolerate

🧩 Groq, AWS Bedrock, Modal, Together AI, Vertex AI, Lambda Labs

---

## Level 1: Evaluation & Telemetry  
**Watch, measure, improve**

Agents don’t fail loudly. They drift. They regress. They “complete” while being wrong. So you need observability that treats an agent run like a distributed system trace.

**What to track**
- Task success rate (not just response quality)
- Tool call accuracy (right tool, right params, right timing)
- Latency breakdown (model vs tools vs retries vs waits)
- Cost per successful outcome (not per token)

**Design questions**
- What does “good” mean for each agent role?
- Where are the control points: before planning, before tool calls, before final output?
- What is your rollback story when a prompt change regresses production?

**Common failure modes**
- No ground truth → “vibes-based” improvement  
- Evaluating only final text → missing tool misuse  
- No replay system → can’t debug or reproduce

🧩 LangSmith, DeepEval, TruLens, Phoenix, Weights & Biases, Arize

---

## Level 2: Core LLM Brains  
**The raw cognition**

This is where capability lives — but capability is not reliability. In multi-agent systems, you’re selecting *behavior under pressure*: ambiguity, partial info, noisy tool results, long workflows, and adversarial inputs.

**Selection criteria that actually matter**
- Instruction-following under tool constraints  
- Long-horizon consistency (can it stick to a plan across 20 steps?)  
- Structured output stability (JSON, schemas, citations, formatting)  
- Refusal quality (safe + helpful, not brittle)

**Design questions**
- Do you need one brain or a mixture (fast + smart)?
- Are you optimizing for planning, writing, coding, extraction, or negotiation?
- What’s your “fallback brain” when the primary fails?

**Common failure modes**
- Over-indexing on benchmark “IQ” and ignoring tool discipline  
- Using one model for everything → expensive and inconsistent  
- No routing → simple tasks burn premium tokens

🧩 GPT-4o, Claude 4, Gemini 2.5, Llama 4, Mistral Large

---

## Level 3: Orchestration Frameworks  
**Assign roles, route messages, coordinate steps**

This is the operating system of your workflow. It turns “a bunch of prompts” into an actual program: branching, retries, state transitions, and role boundaries.

**What orchestration should provide**
- Explicit state (what do we know, what changed, what’s next?)
- Deterministic routing (when do we call which agent?)
- Guarded tool usage (who is allowed to do what?)
- Replay + inspection (debugging is a first-class feature)

**Design questions**
- Is this a graph, a queue, or a conversation?
- When does the workflow stop — and who decides it’s done?
- What’s your strategy for retries: same prompt, new prompt, new model, new agent?

**Common failure modes**
- “Chat” pretending to be architecture  
- No termination conditions → infinite loops  
- Agents stepping on each other → conflicting edits and duplicated work

🧩 LangGraph, CrewAI, AutoGen, Swarm, DSPy Agents, Semantic Kernel

---

## Level 4: Planning Engines  
**Strategic decomposition and goal tracking**

Planning is not “making a checklist.” It’s *constraint-aware* decomposition with progress tracking, verification gates, and graceful backtracking when reality disagrees with the plan.

**Good planning looks like**
- Objectives → subgoals → actions → checks  
- Clear dependencies (what must be true before step 6?)  
- Verification points (prove we’re right, don’t assume)  
- Backtracking rules (what triggers re-plan vs continue?)

**Design questions**
- Do you need global planning or local planning (per step)?
- Are you planning in natural language or structured tasks?
- How do you prevent plan drift when tool outputs surprise you?

**Common failure modes**
- Planning without execution constraints → beautiful fiction  
- No “done definition” → agents keep polishing forever  
- No verification → confident wrongness at scale

🧩 OpenDevin, Voyager Planner, TaskWeaver, MetaGPT, CAMEL, Aide Planner

---

## Level 5: Tool Execution  
**APIs, functions, search — turned into actions**

Tools are where agents touch reality. This is the most fragile layer because it introduces external systems, permissions, latency, schema mismatches, and partial failures.

**What “good tooling” means**
- Stable schemas and strict validation  
- Typed inputs/outputs (your agent shouldn’t guess)  
- Idempotency (safe retries without duplicate side effects)  
- Sandboxing (especially for code execution)

**Design questions**
- Which tools are read-only vs write-capable?
- Who can call what, and under which conditions?
- How do you confirm tool success (don’t trust “200 OK”)?

**Common failure modes**
- Tools returning “almost right” → agent hallucinated glue logic  
- No permission model → accidental destructive actions  
- No guardrails on parameters → subtle, expensive mistakes

🧩 LangChain Tools, OpenAI Functions, E2B, Manifest, ReAct

---

## Level 6: Memory & Context  
**Store thoughts, recall history, think coherently**

Memory is not “dump everything into the prompt.” It’s selective recall: what’s relevant, what’s stable, what must be preserved, and what should expire.

**Types of memory that matter**
- **Working memory:** current task state, constraints, latest tool results  
- **Episodic memory:** what happened in prior runs / similar cases  
- **Semantic memory:** facts, policies, domain knowledge  
- **Preference memory:** style, tone, user-specific rules

**Design questions**
- What should persist across sessions vs expire per run?
- How do you prevent stale memory from poisoning decisions?
- How do you compress context without losing invariants?

**Common failure modes**
- Retrieval noise → irrelevant “memories” derail planning  
- Over-persistence → old constraints applied to new tasks  
- No provenance → you can’t tell where a fact came from

🧩 Zep, Memo, Cognee, Letta, Reverie, PromptLayer

---

## Level 7: Coordination & Messaging  
**Agents talk to each other and the world**

This is the connective tissue: queues, pub/sub, event streams, handoffs, and concurrency control. It’s how you move from “a demo” to “a system.”

**What this layer solves**
- Parallelism (multiple agents working safely at once)  
- Ordering (what must happen before what?)  
- Backpressure (don’t collapse under load)  
- Fan-out/fan-in patterns (researchers → synthesizer)

**Design questions**
- Are agents synchronous (request/response) or event-driven?
- What’s your ordering guarantee: at-most-once, at-least-once, exactly-once?
- How do you prevent race conditions between agents editing the same artifact?

**Common failure modes**
- Duplicate events → duplicated actions  
- No coordination → two agents “fix” the same thing differently  
- No backpressure → tool storms and rate-limit spirals

🧩 Kafka Streams, Redis Pub/Sub, AgentHub, LangServe, Ollama Coordinator

---

## Level 8: Oversight & Governance  
**Humans in the loop, guardrails in place**

This layer turns an agent from “powerful” into “deployable.” It defines what the system is allowed to do, how it gets reviewed, and how it fails safely.

**Governance that’s real (not theater)**
- Role-based permissions (who can execute writes?)  
- Policy enforcement (content, safety, compliance)  
- Approval gates (human sign-off for irreversible actions)  
- Audit logs (who did what, when, and why)

**Design questions**
- What actions require human approval?
- How do you block unsafe tool usage *without* breaking the workflow?
- What’s your incident response when an agent misbehaves?

**Common failure modes**
- Guardrails only at the final output → too late  
- No audit trail → impossible to root-cause  
- Over-restriction → agents become useless and users bypass them

🧩 Guardrails AI, LangFuse, PolicyKit, Conductor, Arize Guard

---

Agentic engineering is entering its systems phase.  
If you're still stitching workflows by feel, you're already behind.

Stop wiring tools and notice the difference.  
Tools change.  
Architectures last.

When you understand the layers, you stop chasing hacks  
And start building systems that scale, adapt, and recover.
