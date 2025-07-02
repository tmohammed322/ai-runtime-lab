<div align="center">

# 🧠 AI Runtime Lab

### Engineering Deterministic Systems Around Non-Deterministic Models

*A hands-on curriculum and reference implementation for building production-grade agentic AI systems — from state machines to multi-agent swarms.*

[![Python](https://img.shields.io/badge/Python-3.11%2B-3776AB?style=flat-square&logo=python&logoColor=white)](https://www.python.org/)
[![License](https://img.shields.io/badge/License-MIT-green?style=flat-square)](#license)
[![Status](https://img.shields.io/badge/Status-Active%20Learning%20Lab-orange?style=flat-square)](#roadmap--progress)
[![Phases](https://img.shields.io/badge/Phases-14%20%2F%2017-blue?style=flat-square)](#-the-14-phases)

[Philosophy](#-core-philosophy) •
[Architecture](#-system-architecture) •
[Phases](#-the-14-phases) •
[Repository Map](#-repository-map) •
[Roadmap](#-roadmap--progress) •
[فارسی](./README.fa.md)

</div>

---

## 📖 About This Repository

This repository is **not** a framework, and it is **not** a wrapper around an LLM API. It is a systems-engineering lab built to answer one question properly:

> **How do you build reliable, auditable, production-grade software on top of a component that is fundamentally probabilistic?**

Every folder here is a self-contained, minimal, runnable proof-of-concept for one architectural concern in agentic AI systems — state control, crash recovery, retries, parallel execution, model routing, edge inference, retrieval, memory, multi-agent orchestration, security, and observability. The code is intentionally small (30–100 lines per concept). The goal is **transferable mental models**, not framework mastery — you should be able to look at any agent framework (LangGraph, Temporal, AutoGen, CrewAI, Restate...) afterward and immediately recognize which of these primitives it is (re)implementing, and what trade-off it made.

This is a personal engineering-education project, structured as a progressive curriculum with runnable reference code at each stop.

---

## 🎯 Core Philosophy

The entire lab is built on a single axiom, referred to throughout as **AI Systems Thinking**:

> **LLMs are probabilistic. Software must be deterministic.**
> Your job as the architect is not to make the model more predictable — it's to build a deterministic runtime *around* an inherently unpredictable component, so the unpredictability is contained, observable, and recoverable.

Left uncontrolled, an LLM-driven agent will:
- Skip steps and hallucinate progress it hasn't actually made
- Get stuck in infinite tool-calling loops
- Lose all state and context the moment a process crashes or the network drops
- Silently return `200 OK` while reasoning has completely derailed

Every phase in this repository exists to close one of these failure modes.

### The four architectural layers

```
                    ┌──────────────────────┐
                    │  AI-NATIVE PRODUCT   │
                    │  & SYSTEM ARCHITECT  │
                    └──────────┬───────────┘
                               │
              ┌────────────────┴────────────────┐
              │                                  │
       GENERATIVE UX                       AI SECURITY
              │                                  │
              └────────────────┬────────────────┘
                               │
                    ┌──────────▼───────────┐
                    │    MODEL ROUTING     │
                    │    EDGE INFERENCE    │
                    └──────────┬───────────┘
                               │
                    ┌──────────▼───────────┐
                    │    AGENT RUNTIME     │
                    │    WORKFLOW ENGINE   │
                    └──────────┬───────────┘
                               │
                    ┌──────────▼───────────┐
                    │ RELIABILITY & STATE  │
                    │ FSM / DURABILITY     │
                    └──────────────────────┘
```

Reliability and state control are the foundation. Nothing above them can be trusted if the base isn't crash-proof and auditable.

---

## 🏗️ System Architecture

The one principle that repeats at every layer of this lab:

> **The LLM never touches the outside world directly.** It only ever *proposes* an intent (a structured, validated suggestion). A deterministic runtime — guarded by an FSM, a tool contract, and a durability layer — is the only thing with permission to actually execute anything.

```
       ┌────────────────────────────────────────────────────────┐
       │                   DETERMINISTIC RUNTIME                │
       │                                                        │
       │   ┌──────────────┐     FSM Guard Check     ┌───────┐   │
       │   │              │ ──────────────────────► │       │   │
       │   │ Context &    │                         │ LLM   │   │
       │   │ State Store  │ ◄────────────────────── │ Engine│   │
       │   └──────────────┘    Structured JSON      └───────┘   │
       │          ▲                                     │       │
       │          │          Durable Activity           │       │
       │          └─────────────────────────────────────┘       │
       │                        Tool Result                     │
       └────────────────────────────────────────────────────────┘
```

This "runtime sandwich" — **FSM guard → validated intent → durable, idempotent execution → checkpoint → observe** — is the pattern every subsequent phase (routing, RAG, memory, multi-agent, security) plugs into.

---

## 🗺️ The 14 Phases

Each phase closes exactly one architectural gap opened by the previous one. They are designed to be read **in order** the first time through.

| # | Phase | Core Question It Answers | Key Concepts |
|---|-------|---------------------------|---------------|
| **1** | [AI Systems Thinking](#phase-1--ai-systems-thinking) | Why do agents go rogue in the first place? | Probabilistic vs. Deterministic |
| **2** | [FSM & State](#phase-2--fsm--state) | How do we stop an agent from skipping or reordering steps? | States, Events, Transitions, Guards, Invariants |
| **3** | [Durable Execution](#phase-3--durable-execution) | How does an agent survive a mid-task crash without redoing (and re-paying for) work? | Workflow vs. Activity, Memoization, Replay |
| **4** | [Failure, Retry & Idempotency](#phase-4--failure-retry--idempotency) | How do we retry safely without duplicating side effects or DDoS-ing our own dependencies? | Backoff + Jitter, Idempotency Keys |
| **5** | [DAG & Workflow Execution](#phase-5--dag--workflow-execution) | How do independent sub-tasks run in parallel instead of burning latency serially? | Fan-out/Fan-in, Immutable Message Passing |
| **6** | [Agent Runtime](#phase-6--agent-runtime) | How do we finally let the LLM make decisions — safely? | Context Manager, Intent Validator, Tool Dispatcher |
| **7** | [Observability & Evals (v1)](#phase-7--observability--evals-v1) | How do we know *why* an agent behaved a certain way, not just *that* it returned 200? | Tracing, Deterministic vs. LLM-as-Judge Evals |
| **8** | [Model Routing](#phase-8--model-routing) | Why pay GPT-4o/Sonnet prices for a task a small model can handle? | Static / Semantic / Cascading Routing, Fallback |
| **9** | [Inference & Edge AI](#phase-9--inference--edge-ai) | How do we run models under a strict RAM/latency budget on real hardware? | Quantization, KV Cache, Memory-Bound Inference |
| **10** | [RAG & Context Management](#phase-10--rag--context-management) | How do we inject external knowledge without drowning the model in noise? | Chunking, Hybrid Search, Re-ranking, RRF |
| **11** | [Memory Systems](#phase-11--memory-systems) | How does an agent remember users and its own past across sessions? | STM/LTM, Semantic vs. Episodic Memory, Consolidation |
| **12** | [Multi-Agent Orchestration & Swarms](#phase-12--multi-agent-orchestration--swarms) | How do specialized agents collaborate without descending into chaos? | Supervisor vs. Peer-to-Peer, Handoff Contracts |
| **13** | [Security, Sandboxing & Safety](#phase-13--security-sandboxing--safety) | How do we stop an agent from becoming a "confused deputy"? | Prompt Injection, Dual-LLM Pattern, AST Sandboxing |
| **14** | [Observability, Tracing & Evaluation (v2)](#phase-14--observability-tracing--evaluation-v2) | How do we monitor a fully-assembled, secured, multi-agent system in production? | OTel Semantic Conventions, Golden Datasets, CI/CD Regression |

> Phases **15–17** (*Generative UX, Distributed AI Systems, Final Architecture Integration*) are part of the broader learning roadmap and are tracked in [Roadmap & Progress](#-roadmap--progress) — implementation folders for these will land in this repository as they are completed.

---

<details>
<summary><h3>Phase 1 — AI Systems Thinking</h3></summary>

Not a folder of code — a worldview. The foundational claim: an LLM is a probabilistic next-token predictor; real software needs deterministic guarantees. Every other phase is a concrete implementation of "wrap the probabilistic core in a deterministic shell." This lens gets revisited at Observability and Security, where new dimensions of the same principle show up.

</details>

<details>
<summary><h3>Phase 2 — FSM & State</h3></summary>

**📁 `fsm/`**

A Finite State Machine acts as a strict commander standing over the LLM: the system exists in exactly one state at a time, and can only move to the next one when a *permitted* event occurs.

| Element | Definition | Example |
|---|---|---|
| **State** | The system's current ground truth | `READING` |
| **Event** | A signal that something happened | `FILE_FOUND` |
| **Transition** | The rule for moving between states | `READING --FILE_FOUND--> ANALYZING` |
| **Invariant** | A rule that must never break, under any circumstance | "No moving to REPORT until file is read" |

Two control layers make the FSM more than a lookup table:
- **Guard** — a boolean condition checked *before* a transition executes (e.g., reject `ANALYZING` if the file read `0` bytes)
- **Invariant** — a hard rule enforced for the entire lifecycle (e.g., "total retries across all states ≤ 3")

An out-of-order event (e.g., `REPORT_GENERATED` arriving while in `READING`) is **rejected** with an exception — the FSM never allows shortcuts or invalid data to slip into the next state.

**Persistence is bolted directly onto the FSM**: every transition is checkpointed to disk *synchronously, before* the next step executes. On reboot, the FSM reads its last known state from disk and resumes exactly where it left off — no wasted re-work, no lost context.

```python
def transition_to(self, new_state: State, guard: bool = True) -> None:
    if new_state not in self.TRANSITIONS[self.state]:
        raise ValueError(f"Illegal transition: {self.state} -> {new_state}")
    if not guard:
        raise PermissionError(f"Guard failed for target state: {new_state}")
    self.state = new_state
    self._checkpoint()  # disk is updated before in-memory state is trusted
```

**Failure semantics at the FSM level:** on error, the system routes to one of three outcomes: `Retry` (transient), `Fail` (deterministic/permanent), or `Escalate` (human-in-the-loop) — the taxonomy fully explored in Phase 4.

</details>

<details>
<summary><h3>Phase 3 — Durable Execution</h3></summary>

**📁 `durable_execution/`**

The problem: a multi-step pipeline (extract → summarize via LLM → save to DB) crashes mid-flight due to a network blip. On restart, a naive re-run calls the LLM *again* — burning tokens, time, and (because LLMs are non-deterministic) potentially getting a **different answer** the second time.

**Durable Execution** (the pattern behind Temporal, Restate) gives your workflow function memory that survives a crash:

> **Golden rule:** on replay, steps that already succeeded are never re-executed — their previous result is loaded from disk and injected back in (Memoization / Replay).

The architecture splits into two strictly separated roles:

- **Workflow (orchestrator)** — must be **100% deterministic**. No `time.now()`, no `random()`, no direct HTTP calls. Just sequencing logic.
- **Activity (executor)** — the only place allowed to touch the outside world (LLM calls, disk I/O, DB writes). Each Activity is independently retried and cached.

```python
def run_activity(step_id: str, func, *args):
    log = json.loads(LOG_FILE.read_text()) if LOG_FILE.exists() else {}
    if step_id in log:
        return log[step_id]              # REPLAY — skip real execution
    result = func(*args)                 # EXECUTE — real, first-time run
    log[step_id] = result
    LOG_FILE.write_text(json.dumps(log, indent=2))
    return result
```

**Why the Workflow/Activity split is non-negotiable:** if you put `datetime.now()` or `random()` directly inside the Workflow body, the decision path diverges on every replay, and the orchestration engine can no longer line up in-flight variables with the events it already logged to disk. That's a silent, hard-to-debug **non-deterministic replay bug** — one of the most common mistakes when building this pattern from scratch.

</details>

<details>
<summary><h3>Phase 4 — Failure, Retry & Idempotency</h3></summary>

**📁 `Failure-Retry-Idempotency/`**

Not all failures are equal. Treating them identically is the first mistake most engineers make:

| Failure Type | Nature | Example | Strategy |
|---|---|---|---|
| **Transient** | Temporary, non-deterministic | `503`, `429 Rate Limit` | Retry with backoff |
| **Permanent** | Deterministic, will never succeed | `401 Auth Failure`, validation error | Fail fast — no retry |
| **Poison Pill** | Input itself causes repeated crashes | Malformed JSON, overflow | Quarantine / escalate |

**Exponential Backoff + Jitter** prevents the *Thundering Herd Problem* — if 10 agents all hit a `429` and all retry after exactly 1 second, they synchronize and take the downstream service down together. Backoff staggers the wait time exponentially (`1s → 2s → 4s → 8s`); jitter randomizes it further so agents desynchronize.

**Idempotency** is the other half of safe retries:

> An operation is idempotent if running it N times produces exactly the same effect as running it once.

Without this, a retried "send email" or "process payment" call can double-fire a side effect that already succeeded — the request went through, but the response was lost to a network drop, triggering a needless retry. The fix is an **Idempotency Key** (e.g., `hash(task_id + step_id)`) attached to every request; if the receiving layer has already processed that key, it returns the cached result instead of re-executing.

```python
sleep_time = random.uniform(0, min(8.0, base_delay * (2 ** attempt)))  # Full Jitter
```

**Closing the deterministic foundation:** Phases 1–4 together produce a *crash-proof runtime* — any LLM can now be mounted on top of it without fear of network drops, financial waste from duplicate side effects, or anarchic step-skipping.

</details>

<details>
<summary><h3>Phase 5 — DAG & Workflow Execution</h3></summary>

**📁 `dag/`**

If FSM is the "compass and lifecycle supervisor," DAG is the "roadmap and parallel-execution engine." A single FSM state (e.g., `EXECUTE`) often hides multiple independent sub-tasks — fetching 5 documents, merging results, running validation code. Running them serially tanks latency; FSM alone has no concept of "run these things concurrently."

A **Directed Acyclic Graph** is built from:
- **Nodes** — executable Activities (API calls, DB queries, text processing)
- **Edges** — data or ordering dependencies between nodes

If `Task_A` and `Task_B` share no dependency, they run fully in **parallel**. `Task_C`, dependent on both, waits for a **fan-in join**. "Acyclic" means no path may loop back to the start node — that would deadlock.

> **Architectural principle:** each node is a **pure function** — read-only input, new output. The DAG engine (not shared mutable state) is responsible for injecting each node's output into the next node's input. Sharing a mutable dict across parallel branches is the single most common way to introduce race conditions here.

**Failure modes specific to DAGs:**
- **Branch isolation** — if one of 3 parallel branches fails, do you `short-circuit` (cancel the whole DAG — for tightly coupled work) or go `best-effort` (push forward with partial data — fine for something like LLM summarization that tolerates 80% coverage)?
- **Static vs. Dynamic DAG** — a static graph is fully known before execution (`Fetch → Process → Save`). A **Dynamic/Agentic DAG**, where the LLM itself decides what nodes to add mid-run, is far harder to manage because the LLM can accidentally introduce a cycle — requiring a runtime guard that detects and rejects cycles on the fly.

```python
ready_nodes = [
    node for name, node in self.nodes.items()
    if name not in completed and node.dependencies.issubset(completed)
]
tasks = [self._run_node(node, results) for node in ready_nodes]
node_results = await asyncio.gather(*tasks)   # true fan-out
```

</details>

<details>
<summary><h3>Phase 6 — Agent Runtime</h3></summary>

**📁 `agent-runtime/`**

With all deterministic layers in place (FSM, Durable Execution, Retry/Idempotency, DAG), Phase 6 is the first time the LLM is mounted as a **decision-maker** on top of this chassis.

An Agent Runtime is the control loop implementing **Observe → Think → Act → Verify** (the ReAct pattern), built from four modules:

- **Context Manager** — assembles system prompt, tool-call history, and current FSM state for the LLM
- **Intent Parser & Validator** — takes the LLM's probabilistic output and forces it through a Pydantic schema
- **FSM Guardrail** — checks whether the LLM's requested action is even legal in the current FSM state
- **Tool Dispatcher** — executes the approved action as a Durable Activity (retryable, idempotent, cacheable)

> **Non-negotiable principle:** the LLM never calls a tool or mutates a database directly. It only ever *proposes* an intent. The Runtime evaluates, logs, and then executes that proposal inside a durable substrate.

The critical failure modes this phase guards against include the **Death Spiral** (an agent stuck endlessly re-calling the same tool) and **Unbounded Autonomy** — both closed off by the combination of FSM guard checks and a hard cap enforced by the Tool Dispatcher.

</details>

<details>
<summary><h3>Phase 7 — Observability & Evals (v1)</h3></summary>

**📁 `Observability-Eval-Engine/`**

Traditional APM (Datadog, Prometheus-style monitoring centered on HTTP status codes, CPU, latency) fails for agentic systems because failures here are **silent**: an API can return `200 OK`, throw zero exceptions, and still have completely hallucinated its answer or burned 3x the expected tokens.

> **Principle:** in AI systems, we don't log code — we trace the *reasoning path* (graph of thought) and *state transitions*.

Two complementary concepts:

| Concept | What It Is |
|---|---|
| **Observability (Tracing)** | Live production monitoring — a `Trace` is the full run; a `Span` is each sub-operation (an LLM call, a tool call) |
| **Evaluations (Evals)** | The replacement for unit tests on non-deterministic behavior — did the output stay correct, faithful, and on-prompt? |

A **two-tier eval engine**:
- **Deterministic Evals** (fast, cheap) — schema validation, latency thresholds, forbidden-word checks
- **Probabilistic Evals / LLM-as-a-Judge** — a separate model scores **Faithfulness** (is the answer grounded in the input data, or invented?) and **Answer Relevance**

**Operational risks to design around:**
- **PII & data leakage in traces** — shipping raw user data/API keys to an external logging system without a redaction layer
- **Goodhart's Law in LLM-as-a-Judge** — optimizing purely for judge-score inflates polish over substance
- **Telemetry overhead** — logging full multi-KB prompts on every call adds real latency and st