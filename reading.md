# Reading List: Closest Prior Work

A practical study guide elaborating on the prior work referenced in `review.md`. For each entry: what it is, how it works, how it relates to the idea in `concept.md`, what to learn from it, and where to read more.

The concept's four pillars, for cross-reference:

- **P1. Recursive decomposition, one layer per cycle** — decompose only one level; defer deeper decomposition; re-assess on updated context.
- **P2. Sub-goal legitimacy challenging + upward feedback** — a sub-goal may question its own validity and trigger recalibration of its parent and siblings (double-loop applied to a goal tree).
- **P3. Goal + achieving activities as an atomic unit** — a goal and its execution lifecycle are managed as one unit with a feedback interface to its parent.
- **P4. Human as judgment source, not approval gate** — human injects taste/judgment to prune the search space, without becoming a bottleneck.

---

## Part A. Directly-tried agent orchestration (the "has this been tried?" cluster)

### 1. BabyAGI

- **What it is.** Proof-of-concept by Yohei Nakajima (2023) demonstrating an objective-driven agent loop with an LLM core.
- **How it works.** Three LLM agents run in a loop against a **flat task list**: (1) a task-creation agent produces new tasks from the previous result, (2) a prioritization agent re-orders the list, (3) an execution agent completes the top task. The objective is fixed at the top and never revised.
- **Relation to concept.** Covers P1 in its weakest form (re-assess → re-prioritize), but the task list is flat, not a tree. No P2 (goals are never questioned), no P3 (no atomic goal unit), no P4 (fully autonomous).
- **What to learn.** This is the closest thing to "people already tried it," and the known failure modes are precisely the ones the concept targets: context loss, no hierarchy, blind re-prioritization, and no mechanism to abandon a bad sub-goal. Reading it shows why the flat version fails on complex goals.
- **Read:** `github.com/yoheinakajima/babyagi`

### 2. AutoGPT

- **What it is.** An autonomous GPT-4 agent (Significant Gravitas, 2023) that chains thoughts, executes commands, and loops until the objective is met. Went viral; ambitious scope; largely abandoned for real work.
- **How it works.** Per-iteration: reasoning → plan → command execution → feedback ingestion, with long-term memory and task management bolted on. Fully autonomous by default.
- **Relation to concept.** Same as BabyAGI: shallow P1, no P2–P4.
- **What to learn.** Demonstrates that autonomy without goal-level feedback drifts: agents commit to elaborate sub-plans and then fail to recognize when the premise is broken. Reinforces the need for P2 and P4.
- **Read:** `github.com/Significant-Gravitas/AutoGPT`

### 3. ADaPT — As-Needed Decomposition and Planning with Language Models

- **What it is.** Prasad et al. (2023). The closest conceptual match to P1.
- **How it works.** Given a task, the model first tries to solve it directly; if the task is judged too complex or the direct attempt fails, it **recursively decomposes into sub-tasks, executes them, and re-aggregates**. Decomposition is invoked *as needed during execution* rather than pre-planned all the way down. Uses "Plan-and-Solve" style self-checking and a terminate-condition to decide when to stop.
- **Relation to concept.** Strongly overlaps P1 (defer decomposition, only go deeper when needed). No P2 (sub-tasks are executed without legitimacy feedback to the parent), no P3, no P4 (no human steering).
- **What to learn.** This is the strongest evidence that the "one layer at a time, decompose only when needed" discipline is tractable and beneficial — it improved performance on complex tasks over pre-planned decomposition baselines. The gap between ADaPT and the concept is exactly P2 and P4.
- **Read:** Prasad, A., Koller, A., Hartmann, M., Clark, P., Sabharwal, A., Bansal, M., & Khot, T. (2023). *ADaPT: As-Needed Decomposition and Planning with Language Models.* Findings of NAACL 2024. arXiv:2311.05772.

### 4. Goal-Driven Autonomy (GDA) / goal-reasoning agents

- **What it is.** A research line in AI planning (Molineaux, Klenk & Aha, 2010; ARTUE; MIDCA) where agents reason about and revise their **own goals** during execution.
- **How it works.** A four-stage loop: a planner generates a plan toward goals → a dispatcher executes → a **monitor** detects discrepancies between expected and observed world state → an explanation/generation module decides whether to replan, revise the current goal, or formulate a new one. Goals are dynamic, not fixed. MIDCA adds a metacognitive layer for self-regulation.
- **Relation to concept.** This is the direct ancestor of P2. The monitor + goal-revision machinery is essentially "sub-goals can be challenged," but (a) it operates on the current plan/goal rather than a recursively-decomposed goal tree, (b) it runs on formal planners rather than LLM agents, and (c) it has no human-steering role.
- **What to learn.** P2 has intellectual precedent and a working pattern (monitor → explain → revise). Porting that pattern into an LLM-based recursive goal tree, with upward propagation to parent goals, is the genuinely under-explored piece.
- **Read:** Molineaux, M., Klenk, M., & Aha, D. W. (2010). *Goal-Driven Autonomy in a Navy Strategy Simulation.* AAAI-10. Also Cox, M. T. (2016). *MIDCA: A Metacognitive, Integrated Dual-Cycle Architecture for Self-Regulated Autonomy.*

### 5. HALO — Hierarchical Autonomous Logic-Oriented Orchestration

- **What it is.** Hou, Tang & Wang (2025). Multi-agent LLM orchestration with a three-level hierarchy.
- **How it works.** A high-level planning agent decomposes the task, mid-level role-design agents instantiate sub-task-specific agents, and low-level inference agents execute. Execution is cast as a structured search (MCTS) over agent actions; an adaptive prompt-refinement module rewrites raw user queries.
- **Relation to concept.** Shares the intuition that complex tasks need hierarchical decomposition for LLM agents. But decomposition is **by role** (planner → designer → executor), not a goal recursing into itself; it is top-down with no upward legitimacy feedback (no P2), no atomic goal unit, and no human steering.
- **What to learn.** Shows the current SOTA direction for hierarchical LLM multi-agent systems and the performance gains available (~14% average improvement). Also shows the limitation: role-based hierarchies don't address goal-validity revision.
- **Read:** Hou, Z., Tang, J., & Wang, Y. (2025). *HALO: Hierarchical Autonomous Logic-Oriented Orchestration for Multi-Agent LLM Systems.* arXiv:2505.13516.

---

## Part B. Structural foundations (hierarchical decomposition)

### 6. HTN Planning (Hierarchical Task Networks)

- **What it is.** A family of automated planners that decompose high-level tasks into sub-tasks until primitive actions are reached (Erol, Hendler & Nau, 1994; SHOP2, Nau et al., 2003).
- **How it works.** A domain library of *methods* maps each task to a partially-ordered set of sub-tasks, each of which is decomposed recursively to primitives. SHOP2 performs *ordered task decomposition*, planning forward from the initial state in execution order.
- **Relation to concept.** Provides the formal backbone for P1 and P3 (a task + its methods is effectively an atomic decomposition unit). But classic HTN decomposes **all the way down in one planning episode**, assuming a stable, known domain — precisely what the concept relaxes for complex, uncertain goals.
- **What to learn.** The mechanics of sound/complete recursive decomposition; useful to know because the concept's "one layer per cycle" is a deliberate deviation from HTN's full decomposition.
- **Read:** Erol, K., Hendler, J., & Nau, D. (1994). *HTN Planning: Complexity and Expressivity.* AAAI-94. Nau, D., et al. (2003). *SHOP2: An HTN Planning System.* JAIR 20, 379–404.

### 7. Options Framework (temporal abstraction in RL)

- **What it is.** Sutton, Precup & Singh (1999). Extends Markov Decision Processes with *options* — closed-loop policies for temporally-extended courses of action.
- **How it works.** An option has an initiation set, an intra-option policy, and a termination condition. Options over an MDP form a semi-Markov decision process, so planning/learning can operate at multiple time scales. Supports *intra-option learning* (learning from fragments of execution) and subgoal-based option shaping.
- **Relation to concept.** Closest formal realization of P3: a goal with its achieving policy treated as a single unit at one level of a hierarchy, with value propagated up. It even supports interrupting/refining an option mid-execution (a form of P2), and shaping options via subgoals.
- **What to learn.** The most rigorous mathematical account of "atomic goal units" and multi-level goals. Reading it clarifies what a principled P3 could look like and where the concept would go beyond it (no human steering, no explicit legitimacy-revision semantics).
- **Read:** Sutton, R. S., Precup, D., & Singh, S. (1999). *Between MDPs and Semi-MDPs: A Framework for Temporal Abstraction in Reinforcement Learning.* Artificial Intelligence 112(1–2), 181–211.

### 8. Continual Planning

- **What it is.** Research area on planning interleaved with execution (desJardins, Durfee, Ortiz & Wolverton, 1999).
- **How it works.** Key ideas: plan refinement as the world is observed; plan repair when execution deviates; **goal reconsideration** — deciding whether current goals are still appropriate; and deciding *how much* to plan given resource costs.
- **Relation to concept.** Directly supports P1's premise (re-assess and re-plan on updated context) and the "optimum planning depth" argument (decide how much to plan, avoid over-commitment). Goal reconsideration overlaps P2, though not in a recursive tree context.
- **What to learn.** The vocabulary for "don't plan too deep" — planning is a continuous resource-bounded activity, not a one-shot artifact. Good grounding for the concept's deferral discipline.
- **Read:** desJardins, M., Durfee, E., Ortiz, C., & Wolverton, M. (1999). *A Survey of Research in Distributed, Continual Planning.* AI Magazine 20(4), 13–22.

---

## Part C. The human / learning loop

### 9. ReAct

- **What it is.** Yao et al. (2023). Interleaves reasoning traces with acting in LLMs.
- **How it works.** The model alternates *thought → action → observation*, updating its action plan as new observations arrive. Reduces hallucination and error propagation versus pure chain-of-thought by grounding reasoning in environment feedback.
- **Relation to concept.** Empirical demonstration of continuous assess/plan/act for LLM agents (the expert-behavior premise). Not hierarchical, no goal revision, no human loop.
- **What to learn.** The baseline mechanism any implementation of the concept would build on for a single goal unit.
- **Read:** Yao, S., Zhao, J., Yu, D., Du, N., Shafran, I., Narasimhan, K., & Cao, Y. (2023). *ReAct: Synergizing Reasoning and Acting in Language Models.* ICLR 2023.

### 10. Reflexion

- **What it is.** Shinn et al. (2023). Adds self-reflection to agents via "verbal reinforcement learning."
- **How it works.** After a failed attempt, the agent writes a verbal reflection into a memory buffer; the next attempt conditions on that reflection, allowing it to learn from mistakes without weight updates.
- **Relation to concept.** A lightweight form of P2 at the *action* level (re-assess past decisions). But reflection is about the agent's own performance, not about questioning the legitimacy of the goal it is pursuing.
- **What to learn.** A concrete, cheap mechanism for "re-assess and adjust" that the concept could reuse inside each goal unit.
- **Read:** Shinn, N., Cassano, F., Gopinath, A., Narasimhan, K., & Yao, S. (2023). *Reflexion: Language Agents with Verbal Reinforcement Learning.* NeurIPS 2023.

### 11. Double-Loop Learning

- **What it is.** Argyris & Schön (1974, 1978, 1996), organizational learning theory.
- **How it works.** *Single-loop* learning corrects a mismatch between intention and outcome without changing governing assumptions. *Double-loop* learning questions and revises the underlying values, goals, and assumptions themselves.
- **Relation to concept.** The conceptual source of P2. The concept's upward feedback — a sub-goal challenging its own legitimacy and recalibrating the parent — is double-loop learning applied across a recursive goal tree.
- **What to learn.** The classic distinction is the theoretical justification for why the concept's "challenge the sub-goal" step is different from ordinary error correction. Also worth reading for the noted *difficulty* of double-loop in practice (defensiveness, oscillation) — directly relevant to the oscillation risk in `review.md` §2.3.
- **Read:** Argyris, C., & Schön, D. A. (1978). *Organizational Learning: A Theory of Action Perspective.* Addison-Wesley.

### 12. Human-in-the-Loop Machine Learning

- **What it is.** Mosqueira-Rey et al. (2023), a state-of-the-art survey of collaboration between humans and ML.
- **How it works.** Taxonomy of paradigms by who controls the loop: active learning (system queries the human), interactive ML, machine teaching (human drives), plus explainable AI and HITL reinforcement learning.
- **Relation to concept.** Grounds P4. Supports the framing that the human supplies information/judgment, not merely approvals, and that interaction should be designed around *what information is worth collecting*.
- **What to learn.** Existing patterns for when/how to query a human, so P4 doesn't re-invent an approval-bottleneck design.
- **Read:** Mosqueira-Rey, E., Hernández-Pereira, E., Alonso-Ríos, D., Bobes-Bascarán, J., & Fernández-Leal, Á. (2023). *Human-in-the-Loop Machine Learning: A State of the Art.* Artificial Intelligence Review 56(4), 3005–3054.

---

## Part D. Conceptual underpinnings

### 13. Bounded Rationality & Satisficing

- **What it is.** Simon (1947, 1955, 1979). Decision-makers face limits on information, computation, and time, so they *satisfice* — accept the first option meeting an acceptable threshold — rather than optimize.
- **Relation to concept.** The formal justification for the "optimum planning depth" argument in `concept.md` §2.3 and the reason exhaustive search by a fully autonomous agent is infeasible (the P4 justification).
- **What to learn.** The canonical framing for why "plan only on certainty, defer the rest" is rational behavior, not a limitation.
- **Read:** Simon, H. A. (1955). *A Behavioral Model of Rational Choice.* QJE 69(1), 99–118. Also (1947) *Administrative Behavior*; (1979) Nobel lecture.

### 14. Situated Action / Plans as Resources

- **What it is.** Suchman (1987). Plans are not blueprints but *resources* for action; actual behavior is improvised in situ.
- **Relation to concept.** Explains why plans made for complex goals get invalidated (context drifts, assumptions fail), motivating both the deferral discipline (P1) and legitimacy challenging (P2).
- **What to learn.** The philosophical grounding for "plans will be wrong; design the process to handle that gracefully."
- **Read:** Suchman, L. A. (1987). *Plans and Situated Actions: The Problem of Human-Machine Communication.* Cambridge University Press.

### 15. OODA Loop

- **What it is.** Boyd (1995). Observe → Orient → Decide → Act, repeated continuously, with orientation at the core.
- **Relation to concept.** The general template for the "assess/plan/act/adjust constantly" premise; the concept applies it recursively at every level of a goal tree.
- **What to learn.** The insight that advantage comes from iterating the loop faster and re-orienting on new information — a good mental model for how frequently a goal unit should re-evaluate.
- **Read:** Boyd, J. R. (1995). *The Essence of Winning and Losing* (unpublished briefing slides).

---

## Part E. Adjacent multi-agent frameworks (worth skimming)

### 16. MetaGPT

- **What it is.** Hong et al. (2023). Multi-agent LLM framework that simulates a software company: roles (PM, architect, engineer) cooperate via standard operating procedures and shared documents.
- **Relation to concept.** Demonstrates multi-agent LLM collaboration in practice, but decomposition follows a fixed role/SOP template, not recursive goal decomposition. No goal revision, no human judgment loop.
- **Read:** Hong, S., et al. (2023). *MetaGPT: Meta Programming for a Multi-Agent Collaborative Framework.* arXiv:2308.00352.

### 17. AutoGen

- **What it is.** Wu et al. (2023), Microsoft. Framework for building LLM applications via multi-agent conversation; supports nested and hierarchical group-chat patterns and human-in-the-loop *intervention*.
- **Relation to concept.** The closest existing *infrastructure* for implementing parts of the concept: nested agents map to a goal tree, and its human-intervention mode maps to P4. Lacks the explicit goal-validity/feedback semantics of P2.
- **Read:** Wu, Q., et al. (2023). *AutoGen: Enabling Next-Gen LLM Applications via Multi-Agent Conversation.* arXiv:2308.08155.

### 18. Voyager

- **What it is.** Wang et al. (2023). Open-ended embodied agent (Minecraft) that builds a *skill library* of reusable, named sub-programs.
- **Relation to concept.** Skills act like reusable atomic goal units (echo of P3 and the options framework), accumulated automatically over tasks. Suggests the concept could benefit from a reusable skill/sub-goal library to avoid re-decomposing the same sub-goals.
- **Read:** Wang, G., et al. (2023). *Voyager: An Open-Ended Embodied Agent with Large Language Models.* arXiv:2305.16291.

### 19. Tree of Thoughts

- **What it is.** Yao et al. (2023). LLM reasoning as search over a tree of intermediate thoughts with evaluation and backtracking.
- **Relation to concept.** Tree *search over reasoning paths* rather than decomposition of goals; relevant as a technique for exploring alternative sub-goal strategies within one level of the concept's tree.
- **Read:** Yao, S., Yu, D., Zhao, J., et al. (2023). *Tree of Thoughts: Deliberate Problem Solving with Large Language Models.* NeurIPS 2023.

---

## Part F. Decomposition domain — adjacent work

Work that decomposes goals/tasks but does not combine as-needed decomposition + upward legitimacy feedback + human steering. Grouped by how they decompose. The takeaway: almost all decompose once up front into a fixed structure, or search within a level; only ADaPT, GDA, ABSTRIPS, and Feudal Networks touch dynamic decomposition in spirit.

### F.1 LLM prompting/planning that decomposes

### 20. Least-to-Most

- **What it is.** Zhou et al. (2022, ICLR 2023). Decompose a problem into a list of subproblems, solve them sequentially, feeding each answer forward as context for the next.
- **Relation to concept.** Early evidence that decomposition + sequential solving beats flat chain-of-thought. But the subproblem list is fixed up front — no re-decomposition, no goal feedback (no P1 dynamism, no P2).
- **Read:** Zhou, D., Schärli, N., Hou, L., Wei, J., Scales, N., Wang, X., Schuurmans, D., Cui, C., Bousquet, O., Le, Q., & Chi, E. (2023). *Least-to-Most Prompting Enables Complex Reasoning in Large Language Models.* ICLR 2023.

### 21. Decomposed Prompting / Successive Prompting

- **What it is.** Khot et al. (2022, EMNLP). Decompose a complex question into modular sub-questions, each answered by a focused sub-prompt, with answers passed down the chain.
- **Relation to concept.** Modular sub-goal structure (echo of P3), but fixed decomposition decided by one LLM call; same AI2 group that later produced ADaPT. No feedback or revision.
- **Read:** Khot, T., Trivedi, H., Bhakthavatsalam, N., & Clark, P. (2022). *Decomposed Prompting: A Modular Approach for Solving Complex Tasks.* EMNLP 2022.

### 22. Plan-and-Solve

- **What it is.** Wang et al. (2023, ACL). Prompt the LLM to first devise an explicit plan, then execute it step by step.
- **Relation to concept.** Decomposition as a distinct planning phase. Plan is generated once and executed; no re-assessment on updated context.
- **Read:** Wang, L., Xu, W., Lan, Y., Hu, Z., Lan, Y., Lee, R. K.-W., & Lim, E.-P. (2023). *Plan-and-Solve Prompting: Overcoming Zero-Shot Chain-of-Thought Reasoning by Large Language Models.* ACL 2023.

### 23. Self-Ask

- **What it is.** Press et al. (2022). LLM asks itself follow-up questions before answering the main one, answering them in sequence.
- **Relation to concept.** Implicit one-level decomposition into sub-questions; static, single-pass.
- **Read:** Press, O., Zhang, M., Min, S., Schmidt, L., Smith, N., & Lewis, M. (2022). *Measuring and Narrowing the Compositionality Gap in Language Models.* (Self-Ask.)

### 24. LLM+P

- **What it is.** Liu et al. (2023). LLM translates the goal into a PDDL planning problem; a classical planner performs decomposition/planning; the LLM executes the plan.
- **Relation to concept.** A different division of labor: delegate decomposition to a sound classical planner rather than the LLM. Plan is complete before execution (opposite of P1's deferral).
- **Read:** Liu, B., Jiang, Y., Zhang, X., Liu, Q., Zhang, S., Biswas, J., & Stone, P. (2023). *LLM+P: Empowering Large Language Models with Optimal Planning Proficiency.* arXiv:2304.11477.

### F.2 Decomposition as search over sub-problem structure

### 25. Graph of Thoughts

- **What it is.** Besta et al. (2024). Generalizes Tree of Thoughts to a graph: thoughts can be merged and branches revisited.
- **Relation to concept.** Allows reworking a branch after feedback — closer in spirit to P2 (challenge/rework) than tree search. Still flat reasoning, not a hierarchical goal tree.
- **Read:** Besta, M., Blach, N., Kubicek, A., Gerstenberger, R., Podstawski, M., Gianinazzi, L., Gajda, J., Lehmann, T., Hoefler, T., & Nyberg, T. (2024). *Graph of Thoughts: Solving Elaborate Problems with Large Language Models.* AAAI 2024.

### 26. LATS — Language Agent Tree Search

- **What it is.** Zhou et al. (2023). Combines MCTS, ReAct, and self-reflection; searches over agent trajectories (reasoning + acting).
- **Relation to concept.** Search across alternative courses of action with reflection; could inform how a goal unit selects among strategies. Operates within a level, not across a recursive goal tree.
- **Read:** Zhou, A., Yan, K., Shlapentokh-Rothman, M., Wang, H., & Wang, Y.-X. (2023). *Language Agent Tree Search Unifies Reasoning Acting and Planning in Language Models.* arXiv:2310.04406.

### 27. RAP — Reasoning as Planning

- **What it is.** Hao et al. (2023). Treats reasoning as planning via MCTS over a world-model's rollouts.
- **Relation to concept.** World-model rollouts could support confidence estimation for a goal unit's deferral/legitimacy decisions (P1/P2), but no explicit goal hierarchy.
- **Read:** Hao, S., Gu, Y., Ma, H., Hong, J., Wang, Z., Wang, D., & Hu, Z. (2023). *Reasoning with Language Model is Planning with World Model.* EMNLP 2023.

### F.3 Multi-agent orchestration that decomposes by role/SOP

### 28. HuggingGPT

- **What it is.** Shen et al. (2023). ChatGPT acts as a controller/planner that decomposes the task and delegates each piece to specialist models in Hugging Face.
- **Relation to concept.** Task decomposition + delegation in practice, but role/delegation based and decided up front; no goal revision, no human loop.
- **Read:** Shen, Y., Song, K., Tan, X., Li, D., Lu, W., & Zhuang, Y. (2023). *HuggingGPT: Solving AI Tasks with ChatGPT and its Friends in Hugging Face.* NeurIPS 2023.

### 29. ChatDev

- **What it is.** Qian et al. (2023). Software development decomposed into a fixed pipeline of communicating agents (design → coding → testing).
- **Relation to concept.** Decomposition as a waterfall SOP, not adaptive; useful as a counter-example of rigid decomposition.
- **Read:** Qian, C., Cong, X., Yang, C., et al. (2023). *ChatDev: Communicative Agents for Software Development.* arXiv:2307.07924.

### 30. AgentVerse / CAMEL

- **What it is.** AgentVerse (Chen et al., 2023): role-specialized agents with an expert-recruitment loop. CAMEL (Li et al., 2023): role-playing agent pairs that decompose tasks via conversation.
- **Relation to concept.** Role-based collaboration rather than recursive goal decomposition; no upward goal feedback.
- **Read:** Chen, W., Su, Y., Zuo, J., et al. (2023). *AgentVerse: Facilitating Multi-Agent Collaboration and Exploring Emergent Behaviors.* ICLR 2024. Li, G., Hammoud, H., Itani, H., Khizbullin, D., & Ghanem, B. (2023). *CAMEL: Communicative Agents for "Mind" Exploration of Large Language Model Society.* NeurIPS 2023.

### F.4 Classical hierarchical decomposition (the foundations)

### 31. ABSTRIPS

- **What it is.** Sacerdoti (1974). Planning in a hierarchy of abstraction spaces: solve at a high abstraction level first, refine into lower-level detail later.
- **Relation to concept.** The original "plan at high level, defer lower-level refinement" idea — a direct ancestor of P1's one-layer-at-a-time discipline.
- **Read:** Sacerdoti, E. D. (1974). *Planning in a Hierarchy of Abstraction Spaces.* Artificial Intelligence 5(2), 115–135.

### 32. Feudal Networks (hierarchical RL)

- **What it is.** Vezhnevets et al. (2017, ICML). Manager/worker hierarchy: manager sets sub-goals for workers; rewards are delegated down the hierarchy.
- **Relation to concept.** A working multi-level goal hierarchy with top-down goal-setting (echo of P3). But the direction is downward only — no upward legitimacy feedback from workers to manager (no P2).
- **Read:** Vezhnevets, A. S., Osindero, S., Schaul, T., Heess, N., Jaderberg, M., Silver, D., & Kavukcuoglu, K. (2017). *FeUdal Networks for Hierarchical Reinforcement Learning.* ICML 2017.

### 33. Work Breakdown Structure (WBS)

- **What it is.** Project-management practice (PMI). Recursively decompose a project into deliverables/sub-deliverables until work packages are estimable and assignable.
- **Relation to concept.** The battle-tested "boring" instance of recursive goal decomposition; worth knowing as the mature analog of the concept's goal tree, including its failure mode (WBS is static and rarely re-validated against changing context).
- **Read:** Project Management Institute. (2021). *A Guide to the Project Management Body of Knowledge (PMBOK Guide)* (7th ed.).

---

## Summary: mapping prior work to the four pillars

| Prior work | P1 (one layer / defer) | P2 (sub-goal legitimacy + upward feedback) | P3 (atomic goal unit) | P4 (human judgment, not approval) |
|---|---|---|---|---|
| BabyAGI / AutoGPT | weak (flat, re-prioritize) | — | — | — |
| ADaPT | strong | — | partial | — |
| Goal-Driven Autonomy | partial | strong (at plan level) | — | — |
| HALO | partial (role hierarchy) | — | — | — |
| HTN / SHOP2 | opposite (full decomposition) | — | strong (task+methods) | — |
| Options framework | partial | partial (option interruption) | strong | — |
| Continual planning | strong | partial (goal reconsideration) | — | — |
| ReAct / Reflexion | partial | partial (self-reflection) | — | — |
| Double-loop learning | — | strong (conceptual source) | — | — |
| HITL-ML | — | — | — | strong |
| Bounded rationality | strong (rationale) | — | — | strong (rationale) |
| Situated action | strong (rationale) | strong (rationale) | — | — |
| Least-to-Most / Decomposed Prompting / Plan-and-Solve / Self-Ask | fixed up-front decomposition | — | partial | — |
| LLM+P | opposite (planner does full decomposition) | — | — | — |
| Graph of Thoughts / LATS / RAP | search within a level | partial (rework branches) | — | — |
| HuggingGPT / ChatDev / AgentVerse / CAMEL | role/SOP-based | — | — | — |
| ABSTRIPS | strong (ancestor of deferral) | — | — | — |
| Feudal Networks | strong (multi-level hierarchy) | — | strong | — |
| WBS | strong (static) | — | strong | — |

**Reading order suggestion.** Start with ADaPT and Goal-Driven Autonomy (closest matches to P1/P2), then the options framework (P3), then the HITL survey (P4). Skim BabyAGI/AutoGPT to internalize the known failure modes. Use HTN, continual planning, and the conceptual pieces (Simon, Suchman, Boyd, Argyris & Schön) as background for the rationale. For the decomposition domain specifically, ABSTRIPS and Least-to-Most show the ancestor and the common fixed-decomposition pattern; Graph of Thoughts/LATS show in-level search; Feudal Networks and WBS show multi-level goal hierarchies without upward feedback.
