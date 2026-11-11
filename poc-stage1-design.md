# POC Stage 1 Design — Autonomous Arm, mini-swe-agent Base Harness

Companion to `concept.md` (Section 6, Stage 1). This document specifies how to build and run the Stage 1 autonomous experiment that tests **H1** (upward legitimacy feedback reduces wasted work) using **mini-swe-agent** as the base single-agent harness on **SWE-Bench-Lite**.

Status: draft design. Scope: the buildable experiment, not the research rationale (see `concept.md` and `reading.md`).

---

## 1. Objective

Validate **H1**:

> On a goal with ≥ 3 decomposition levels, allowing sub-goals to challenge their own legitimacy and propagate feedback to the parent reduces wasted agent steps (tokens, tool calls, dead-end sub-goal time) compared to a baseline that grinds on fixed sub-goals.

The decisive comparison is **C (as-needed decomposition, no feedback) vs. D (as-needed decomposition + upward legitimacy feedback)**, with factors locked.

## 2. Relationship to concept.md

| Stage 1 element | concept.md reference |
|---|---|
| Tested claim | H1 (only open autonomous claim) |
| Conditions A–D | Section 6, Stage 1 |
| B vs. C | Harness sanity check for accepted ADaPT premise (H2) |
| C vs. D | Isolates P2 → H1 |
| Metrics | Section 6, Metrics |
| Domain | "multi-component software/ops tasks" → SWE-Bench-Lite chosen here |

## 3. Domain and Task Selection

### 3.1 Domain: SWE-Bench-Lite

- **Why it fits.** Long feedback cycles (must read repo, edit, run tests), multiple decomposition levels (understand → locate → fix → validate), verifiable outcomes (test suite), and observable wasted work (tokens/steps on abandoned sub-goals).
- **Why SWE-Bench-Lite (300 instances) rather than full SWE-Bench (2294).** Smaller, curated, and the standard target for agentic coding harnesses; fits the ~10–30-goal budget in concept.md.

### 3.2 Subsampling

- Sample **N = 24 instances per condition** (4 conditions → 96 runs) from SWE-Bench-Lite, stratified to cover a mix of difficulty and repository diversity.
- Hand-verify each sampled instance is resolvable (has a known passing patch) and that the repo builds/installs in the sandbox.
- **Fixed goal set across all conditions**: the same 24 instances, in the same order, same seeds.

### 3.3 Ground truth

- Patch correctness judged by the official `swebench` evaluation harness: a patch passes if it resolves the instance's FAIL_TO_PASS tests without breaking PASS_TO_PASS tests.
- Ground-truth decomposition is **not** required — the harness records the agent's decomposition tree, and metrics are computed from the trace, not from a reference tree.

## 4. Architecture

```
┌─────────────────────────────────────────────────────────────┐
│ Orchestration Harness  (the layer under test)               │
│  - GoalTreeManager (create/update/abandon goal nodes)       │
│  - Decomposer       (P1: one-level-as-needed decomposition) │
│  - Challenger       (P2: legitimacy check)  [condition D]   │
│  - Recalibrator     (P2: parent recalibration) [condition D]│
│  - TraceRecorder    (instrumentation / metrics)             │
├─────────────────────────────────────────────────────────────┤
│ Base Harness: mini-swe-agent (locked, shared by all conds)  │
│  - agent loop: observe → think → act                        │
│  - vanilla tools: shell, file read/write, search            │
├─────────────────────────────────────────────────────────────┤
│ Sandbox: per-instance Docker container with repo + tests    │
└─────────────────────────────────────────────────────────────┘
```

- **The contribution is the orchestration layer.** mini-swe-agent supplies the low-level agent loop and tools; it is a black-box primitive shared identically across A–D.
- The orchestration layer intercepts the agent's objective: instead of a fixed single objective, the agent's active objective is the current **goal node**; the harness decides when to enter/exit decomposition and when to re-point the agent at a revised goal.

## 5. Goal Model

A goal node `G`:

| Field | Description |
|---|---|
| `spec` | natural-language goal statement |
| `parent` | parent node or `null` (root) |
| `children` | sub-goals created by decomposition |
| `state` | `pending \| active \| achieved \| abandoned \| revised` |
| `premises` | assumptions the sub-goal rests on (recorded at creation) |
| `context` | accumulated evidence (file states, test results, observations) |
| `cost` | tokens / tool calls / wall-time spent in this node |
| `challenge_count` | number of legitimacy challenges fired (for D) |

Root goal per instance: *"Resolve issue #<id> in repo <name>."*

## 6. Orchestration Harness Components

### 6.1 Decomposer (P1 — active in C and D)

- Trigger: when a goal node becomes `active` and is judged non-primitive.
- Behavior: an LLM module (same model as base agent) takes `spec + premises + context` and outputs either:
  - **"attempt directly"** (goal is primitive — run mini-swe-agent loop until it declares done or stalls), or
  - **"decompose into ≤ 4 sub-goals"** (one level only; sub-goals are created as `pending` children; further decomposition is deferred and re-assessed when a child becomes `active`).
- Constraint: **strictly one level per cycle.** A newly created sub-goal does not auto-decompose; it first evaluates its own premises on activation.
- Stalling/retry: if the base agent makes no progress (no tool calls / repeated same action) for a threshold, the node returns to the decomposer for re-assessment.

### 6.2 Challenger (P2 — active in D only)

- Trigger: periodically — after each sub-goal completes (`achieved`), and on stall (no-progress threshold).
- Behavior: LLM module evaluates the node against its recorded `premises`:
  - **"legitimate"** → continue.
  - **"challenge"** → emit a structured challenge: `{goal_id, violated_premise, evidence, proposed_action}` where proposed_action ∈ {abandon, revise, re-decompose, escalate}.
- Anti-oscillation: a challenge only fires if confidence ≥ threshold AND `challenge_count` < per-node cap AND the parent hasn't recalibrated the same node in the last cooldown window (see §9.3).

### 6.3 Recalibrator (P2 — active in D only)

- Trigger: on receiving a challenge from a child.
- Behavior: the parent node evaluates the challenge and chooses one of:
  - **accept** → apply proposed action to the child (abandon/revise/re-decompose), updating `state`; cascades to siblings only if the parent itself changes.
  - **reject** → leave child unchanged; log the decision.
- Every recalibration event is recorded: `{parent_id, child_id, challenge, decision, rationale, cost_delta}`.

### 6.4 TraceRecorder (all conditions)

Records for every run: the full goal tree with node states and costs, per-node token/tool-call counts, challenge events (D), recalibration events (D), and the final patch. This is the source of all metrics.

## 7. Condition Configurations

Same harness binary; conditions are parameter switches.

| Cond | Tree structure | Decomposition | Legitimacy feedback | Base agent | Purpose |
|---|---|---|---|---|---|
| A | flat task list | full-ish, one-shot up front; re-prioritize after each item | off | mini-swe-agent | baseline |
| B | full tree (all levels up front) | eager full decomposition, then execute in order | off | mini-swe-agent | sanity check vs. C (ADaPT premise) |
| C | tree, built one level at a time | as-needed (Decomposer on) | off (Challenger off) | mini-swe-agent | P1 only |
| D | tree, built one level at a time | as-needed (Decomposer on) | on (Challenger + Recalibrator on) | mini-swe-agent | P1 + P2 → **tests H1** |

Notes:
- A = BabyAGI-style flat loop: create task list from root goal → execute → create/prioritize next tasks from the result. No parent-child feedback.
- B = HTN-style: decompose the whole goal up front to near-primitive leaves, then execute leaves in dependency order.
- A, B, C, D share the exact same mini-swe-agent loop, tools, model, sandbox, and metrics instrumentation. Only the orchestration parameters differ.

## 8. Instrumentation and Metrics

### 8.1 Primary (H1 — wasted work)

- **Wasted tokens** = tokens spent inside nodes that are later `abandoned` or `revised` (cumulative, per instance).
- **Wasted tool calls** = tool calls inside abandoned/revised nodes.
- **Abandoned sub-goal fraction** = abandoned nodes / total created nodes.
- **Cost to completion** = total tokens and wall-time per completed instance.

### 8.2 Secondary (quality / sanity)

- **Resolution rate** = % instances with a passing patch (official swebench eval).
- **Pass@patch** quality of produced patches.
- **C vs. B sanity check**: C should beat or match B on resolution/cost (reproduces ADaPT's as-needed > full-pre-decomposition result). If not, the harness is suspect and H1 results are untrusted.

### 8.3 Failure analysis (per instance)

- For D: log every challenge, its proposed action, and the recalibration decision; tag each as *helped / no-op / harmful* based on downstream cost change.
- For oscillation detection: count repeated challenge→reject→re-challenge cycles on the same node.

### 8.4 Locked factors

- Same model (e.g., GPT-4o / a single fixed model), temperature, and base-agent prompt config across A–D.
- Same 24 instances, same order, same sandbox images.
- Decomposer/Challenger/Recalibrator use the same model as the base agent (no privileged model).

## 9. Experimental Protocol

1. **Bootstrap.** Pick 24 instances; verify each resolves in the sandbox and has a known-good patch.
2. **Sanity gates.**
   - Run B and C on a pilot of 4 instances; confirm C ≥ B (ADaPT premise reproduces) before spending the full budget.
   - Confirm A is strictly worse on cost than C (otherwise the hierarchy adds no value).
3. **Full runs.** Run A–D on all 24 instances. One run = one instance + one condition. Runs are deterministic seeds where possible; where not, run each (instance, condition) with a fixed seed and record variance over 3 seeds for the primary comparison.
4. **Compute.** For each condition: per-instance and aggregate wasted-work metrics + resolution rate; paired comparison C vs. D on the primary metric (this is H1's test).
5. **Report.** Include the sanity-check results (B vs. C), the H1 comparison (C vs. D), and the failure-analysis tags.

## 10. Risks and Mitigations

| Risk | Mitigation |
|---|---|
| Decomposer/Challenger LLM calls add overhead, inflating D's cost and confounding "wasted work" | Measure overhead separately; report wasted work excluding orchestration-call tokens, and report orchestration overhead explicitly |
| Challenge→reject→re-challenge oscillation | Confidence threshold, per-node challenge cap, cooldown window (§6.2, §6.3) |
| "Wasted work" undefined for conditions without trees (A) | For A, approximate abandoned work = tokens spent on flat-list tasks later dropped by re-prioritization |
| Base-agent variation across conditions | Base loop/tools/model identical; only orchestration params differ |
| SWE-Bench-Lite patch size too small to exercise deep decomposition | Pilot on instances; if most are single-edit, select harder instances (multi-file / multi-step) or widen to a subset of full SWE-Bench |
| Orchestration-call tokens dominate (LLM used for every decompose/check) | Cap orchestration LLM calls per node; use cheap scoring (e.g., logit-level or rule-based legitimacy checks where possible) |
| Reproducibility of mini-swe-agent itself | Pin commit, lock tool set and prompts; record exact version in the report |

## 11. Deliverables / Milestones

1. **M0 — Harness scaffold** (1–2 days): orchestration layer wiring; goal-tree manager; TraceRecorder.
2. **M1 — Condition switches** (1–2 days): A/B/C/D parameterization; Decomposer.
3. **M2 — P2 modules** (2–3 days): Challenger + Recalibrator with anti-oscillation.
4. **M3 — Pilot sanity gates** (1 day): B vs. C and A vs. C on 4 instances.
5. **M4 — Full runs** (budget-bound): 96 runs + seeds.
6. **M5 — Analysis + report**: H1 verdict, sanity check, failure analysis.

## 12. Open Questions

- Exact tool set of the installed mini-swe-agent version (confirm from its repo before M0).
- How to define "no-progress threshold" and "primitive goal" for SWE instances (pilot will tune).
- Whether SWE-Bench-Lite instances yield ≥ 3 effective decomposition levels; if not, whether to widen the instance selection.
- Whether orchestration decisions (decompose/challenge) should use the same model as the base agent or a cheaper one (cost trade-off; default to same model for Stage 1 cleanliness).
