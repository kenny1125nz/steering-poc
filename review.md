# Concept Review

Review of the idea described in `concept.md` — recursive goal decomposition based steerable agent orchestration.

## 1. Novelty Assessment

### 1.1 Individual Components

Each building block of the idea already exists in prior work:

| Concept | Closest prior work |
|---|---|
| Recursive decomposition into sub-goals | HTN planning (Erol, Nau, Hendler 1994; Nau et al. 2003) |
| Interleaved assess/plan/act cycles | OODA loop (Boyd), ReAct (Yao et al. 2023), continual planning |
| Don't plan too deep, re-assess on updated context | Bounded rationality / satisficing (Simon 1947, 1955); anytime planning (Boddy & Dean 1989) |
| Plans may embed errors discovered only during execution | Situated action (Suchman 1987); plans as resources, not blueprints |
| Challenging goal assumptions during execution | Double-loop learning (Argyris & Schön); goal-driven autonomy / goal reasoning agents (Cox, Aha, Munoz-Avila) |
| Human-in-the-loop | Mature field (Mosqueira-Rey et al. 2023 survey) |
| Multi-agent LLM orchestration with hierarchical planning | HALO (Hou et al. 2025), MetaGPT, AutoGen |
| Flat task decomposition → execute → re-plan | BabyAGI (2023), AutoGPT |

### 1.2 What Feels Novel

The **synthesis** of these elements into a single orchestration discipline is where the contribution lies. Three aspects appear under-explored:

1. **One-layer-at-a-time decomposition with deferred depth.** HTN planners decompose all the way to primitives in one session. BabyAGI/AutoGPT use flat task lists, not trees. The explicit discipline of "decompose only one level, defer the rest, re-assess" is not formalized as a core orchestration principle in existing work.

2. **Structured upward legitimacy-feedback from sub-goal to parent.** Goal-driven autonomy and goal reasoning agents do something similar in spirit — detecting discrepancies and modifying goals — but the specific protocol of a sub-goal explicitly challenging its own legitimacy and triggering recalibration of its parent and siblings is distinct from existing frameworks.

3. **Human as judgment-source (not approval gate), positioned to reduce search space.** Not entirely new framing (good HITL design avoids approval bottlenecks), but its integration into the recursive decomposition model is not seen articulated this way.

### 1.3 Bottom Line

Not a fundamental breakthrough where no one has thought of anything similar. Goal-driven autonomy, continual planning, HTN, and HITL all cover pieces of the ground. But the specific combination — recursive single-layer decomposition + upward sub-goal challenging + human as judgment source — is a coherent and reasonably original design contribution. A formal literature review (comparing against HALO, BabyAGI, goal-driven autonomy, ADaPT) would be needed to fully settle novelty.

## 2. POC Feasibility

### 2.1 Has This Been Tried?

**Not in the specific combination proposed.**

- **BabyAGI / AutoGPT (2023)** — the viral "autonomous agents" — did the flat "decompose goals → prioritize → execute → re-prioritize" loop. They failed on complex goals for exactly the reasons this design targets: over-committing to plans, no mechanism to challenge bad sub-goals, and full autonomy leading to drift and loops. People largely abandoned them for serious work.
- **ADaPT (Prasad et al., 2023)** — closest conceptual match to "one layer at a time, only decompose further when needed." Recursively decomposes tasks as-needed during execution rather than pre-planning everything. But it has no upward feedback mechanism and no human steering.
- **Goal-Driven Autonomy** (classical AI planning) — goals have a lifecycle, a monitor detects discrepancies, and goals can be revised or abandoned during execution. The "challenge sub-goal legitimacy" concept exists here. But GDA uses formal planners, not LLM-based agents, and has no human-in-the-loop framework.
- **HALO (2025)** — hierarchical multi-agent LLM orchestration, but decomposition is by role (planning agent → role-design agents → execution agents), not by a goal recursing into itself with feedback.

### 2.2 Key Difference From What Exists

- Flat task-list agents (BabyAGI) tried the general approach and **failed** on complexity.
- Formal hierarchical planners (HTN, GDA) have the structure but not the LLM flexibility.
- The proposal sits in the gap between them: hierarchical structure + LLM reasoning + one-layer-at-a-time discipline + upward legitimacy feedback + human steering.

### 2.3 Would It Work?

The **core insight that sub-goals should challenge their own legitimacy** is the most practically valuable piece and the least explored. Most agent systems, once they create a sub-goal, just grind on it. Building a mechanism that periodically asks "does this sub-goal still make sense given what we've learned?" would avoid a lot of wasted work, even if nothing else is built.

**Main practical risk: oscillation.** A sub-goal challenges its parent, the parent recalibrates, a sibling changes, which cascades changes back. Needs dampening or a minimum-confidence threshold for triggering recalibration.

### 2.4 Recommendation

It hasn't been tried in this form. The closest attempt (BabyAGI) failed, and this design addresses the known failure modes. Worth a POC.
