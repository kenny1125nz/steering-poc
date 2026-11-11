# Recursive Goal Decomposition for Steerable Agent Orchestration — Engineering Design Document

## Status

Draft design doc for an engineering experiment / technical spike. Not a research proposal. Novelty is not claimed; the document's purpose is to define a buildable system, identify what to validate, and specify how to validate it. Supporting analysis lives in `review.md` (novelty + feasibility) and `reading.md` (build-vs-borrow study guide). The academic framing of the same idea lives in `concept.md`.

---

## 1. Problem Statement

LLM-based agents that mirror expert practice — continuous assess/plan/act/adjust cycles — perform adequately on small or well-specified tasks but degrade on complex goals. Known failure modes, observed in flat task-list agents (BabyAGI, AutoGPT):

- **Over-commitment to plans** made before enough is known; plans built on assumptions that later fail.
- **No mechanism to question a sub-goal** once created; the agent grinds on goals whose premises have been invalidated.
- **Context drift** — long feedback cycles mean the situation at execution time differs from the situation at planning time.
- **Uncertainty compounding across layers** — an error at an upper decomposition layer invalidates every lower-layer commitment built on it.

The engineering question is not "can this be done" but: **does a structured orchestration discipline that addresses these failure modes reduce wasted work and improve outcomes on complex goals, relative to existing off-the-shelf agent approaches?**

## 2. Goal / Non-Goals

### Goals

- Build an orchestration harness that manages complex goals as a recursive tree of goal units.
- Validate the one mechanism that existing systems lack: sub-goal legitimacy feedback with upward propagation (Section 5, H1).
- Keep the human in the loop as a source of judgment/taste that prunes the search space, without becoming an approval bottleneck.

### Non-goals

- Not a new AI technique, planning algorithm, or model. All reasoning components are existing LLM machinery.
- Not a general-purpose agent framework (no attempt to replace AutoGen, LangGraph, etc.).
- Not a claim of generality across domains; the experiment targets one domain class.
- Not a novelty claim; prior work is treated as components to reuse, not gaps to defend.

## 3. Approach

An orchestration harness in which:

1. A complex goal is decomposed into sub-goals recursively, but **strictly one level per planning cycle**. Deeper decomposition is deferred and evaluated as part of the sub-goal's own achieving activities (ADaPT-style as-needed decomposition).
2. Each goal and its achieving activities are modeled as a single **atomic unit** — a goal specification, context, execution state, and a feedback interface to its parent (echoes the options framework / HTN task+methods).
3. As part of achieving a sub-goal, the unit may **challenge the legitimacy of the sub-goal itself**. If execution evidence indicates the sub-goal's assumptions are violated, it issues feedback upward, triggering recalibration of the parent and siblings (double-loop learning applied to a goal tree; goal-driven autonomy pattern).
4. The human is engaged at structured points to inject judgment/taste/domain knowledge — to prune the search space — not to approve routine actions.

## 4. Build-vs-Borrow

The design is an assembly of existing components plus one novel mechanism. `reading.md` details each; the summary:

| Component | Borrow from | Status |
|---|---|---|
| Recursive as-needed decomposition (P1) | ADaPT, HTN/SHOP2 decomposition discipline | Existing — reuse |
| Atomic goal unit with feedback interface (P3) | Options framework; HTN task+methods | Existing — reuse |
| Interleaved reasoning/acting per unit | ReAct / Reflexion | Existing — reuse |
| Self-reflection on performance | Reflexion | Existing — reuse |
| Human-in-the-loop interaction patterns | HITL-ML literature (Mosqueira-Rey et al.) | Existing — reuse |
| Nested-agent infrastructure | AutoGen (nested group chat), LangGraph | Existing — reuse |
| Sub-goal legitimacy challenge + upward propagation (P2) | Only partially covered (GDA at plan level, not goal tree; no LLM+upward semantics) | **Build — the experiment** |

## 5. Hypotheses and Success Criteria

Only open claims appear here. The deferred-decomposition premise (ADaPT's finding) is accepted as true from prior work and is used as a harness sanity check in Section 6, not as a hypothesis.

### H1 — Upward legitimacy feedback reduces wasted work (primary)

> On a goal with ≥ 3 decomposition levels, allowing sub-goals to challenge their own legitimacy and propagate feedback to the parent reduces wasted agent steps (tokens, tool calls, dead-end sub-goal time) compared to a baseline that grinds on fixed sub-goals.

- Metric: total agent cost (steps/tokens) to reach a completed goal; fraction of completed sub-goals later abandoned.
- Pass: measurable reduction in wasted work on ≥ N test goals (N defined in the experiment plan).

### H2 — Human steering improves outcome without bottlenecking

> Structured human judgment inputs (on goal priorities and search-direction choices) improve outcome quality and/or reduce cost, while total human attention stays bounded (e.g., ≤ X interventions per goal).

- Metric: outcome quality score (task-defined) and human intervention count/time.
- Accept: quality not worse than autonomous baseline at bounded human cost.

## 6. Experiment Design

Two stages. Stage 1 is fully autonomous and isolates H1 (the only open claim) with one factor varied at a time; condition B doubles as a harness sanity check for the accepted deferred-decomposition premise. Stage 2 is a separate experiment for H2 (human-in-the-loop) that **locks the winning autonomous configuration and varies only human presence**, so the human's effect is not confounded with P1/P2.

### Stage 1 — Autonomous arm (no human)

Tests H1. Everything except the factor under test is locked across conditions: same goal set, same underlying agent machinery (ReAct/Reflexion loop), same tools, same infrastructure.

**Scope**

- One domain class, chosen for: long feedback cycles, multiple decomposition levels, verifiable outcomes, observable "wasted work." Candidate: multi-component software/ops tasks (e.g., building a system with several dependent modules) or research-planning tasks. Pick one for v1.
- Goal count: enough for statistical signal but manageable to hand-verify (target ~10–30 per condition).

**Conditions (one factor changed per step)**

1. **A — Flat agent** (BabyAGI-style): decompose → execute → re-prioritize; no hierarchy, no goal feedback. Baseline.
2. **B — Full pre-decomposition** (HTN-style): decompose all the way down up front, then execute. **Sanity-check control** — B vs. C should reproduce ADaPT's known result (accepted premise; see Section 5 intro). This validates the harness, not a claim; B vs. C is a config change, so it costs a single extra condition run and also yields a domain-specific effect size.
3. **C — As-needed decomposition only** (ADaPT-style): P1, no P2. B vs. C is the harness sanity check; C vs. D isolates P2 → **H1**.
4. **D — P1 + P2**: as-needed decomposition + upward legitimacy feedback; no human. C vs. D isolates P2 → **H1**.

Metrics, cost, and failure analysis (below) are identical across A–D.

### Stage 2 — Human arm (separate experiment, other factors locked)

Tests H2. The human-in-the-loop is evaluated as its own experiment so its effect is attributable:

- **Locked factors:** the best autonomous configuration from Stage 1 (expected: D), the same goal set, the same machinery. No other factor changes.
- **Varied factor:** only human presence — Condition **E = locked config + human steering** vs. Condition **D = locked config, no human** (re-run from Stage 1 under identical setup as the reference).
- **Human policy must be fixed across runs** for comparability (same human/role-played scripted policy, same intervention protocol and UI), so that what varies is human presence, not ad-hoc variation in how the human intervenes.

If E beats D, the gain is attributable to the human factor; if not, H2 fails cleanly without contaminating the H1 results.

### Metrics

- Primary: wasted-work proxy (agent steps/tokens, abandoned sub-goals), completion rate, cost-to-completion.
- Secondary: outcome quality, human intervention count/time.
- Failure analysis: log where upward feedback triggered (Stage 1) and where human interventions occurred (Stage 2); record whether recalibration/intervention helped or caused oscillation.

## 7. Risks and Mitigations

| Risk | Mitigation |
|---|---|
| **Oscillation / thrashing**: sub-goal challenges parent → sibling changes → cascade loops | Minimum-confidence threshold before triggering recalibration; cooldown/lockout per unit; max recalibration depth per cycle (from double-loop learning literature on the difficulty of goal revision) |
| **"One level" is ill-defined / domain-dependent** | Make decomposition depth a parameter; define a level as "a unit the executing agent can attempt directly"; tune per domain |
| **Human becomes the bottleneck anyway** | Timebox human interventions; only surface decisions with high leverage (goal priorities, search direction), not routine steps (H2 bounds it) |
| **Baseline unfairness / effort inflation** | Reuse off-the-shelf AutoGen/LangGraph patterns for baselines; share as much machinery as possible |
| **Confounds with domain difficulty** | Use a single domain for v1; multiple goals within it; report per-goal variance |

## 8. Open Questions

- How to define and measure "one level of decomposition" objectively.
- Confidence thresholds for (a) deferring deeper decomposition and (b) triggering a legitimacy challenge.
- Propagation semantics: should a legitimacy challenge always cascade to siblings, or only on parent recalibration?
- Reusable sub-goal/skill library (Voyager-style) — when is a sub-goal worth caching rather than re-decomposing?
- Interaction design for human steering: what to surface, when, and in what form.

## 9. Related Documents

- `concept.md` — academic framing of the idea (original concept paper).
- `review.md` — novelty and feasibility assessment; positions this as an engineering experiment, not a research contribution.
- `reading.md` — annotated study guide on the closest prior work (BabyAGI, AutoGPT, ADaPT, GDA, HALO, HTN, options, continual planning, ReAct/Reflexion, double-loop learning, HITL, and adjacent frameworks).
