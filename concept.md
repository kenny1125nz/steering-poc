# Recursive Goal Decomposition for Steerable Agent Orchestration: A Concept Paper

## Abstract

Large language model (LLM) agents are increasingly tasked with complex, multi-step goals. Existing agent architectures typically either pre-plan a full decomposition of a goal and then execute, or interleave planning and execution with only local, single-loop feedback. Neither approach fully reflects how skilled humans actually manage complex goals: continuously reassessing priorities, questioning assumptions behind current objectives, and revising plans in response to new evidence. This paper proposes a framework for **steerable agent orchestration** based on recursive goal decomposition with upward legitimacy feedback. Goals are decomposed into sub-goals recursively; each sub-goal is treated as an atomic unit with its own achieving activities; and — critically — a sub-goal may **challenge its own legitimacy** based on execution evidence, propagating feedback upward to trigger recalibration of the parent goal. A human operates as a source of judgment and domain knowledge at structured decision points rather than as an approval gate. We describe the design principles, a concrete architecture, and the anticipated failure modes and mitigations. The framework is grounded in prior work on expert practice (Schön, Dreyfus & Dreyfus, Simon), planning architectures (HTN, options framework, ADaPT), and double-loop learning (Argyris & Schön). We argue that the combination of one-layer-at-a-time decomposition, upward legitimacy feedback, and judgment-based human steering is under-explored in current agent design and merits empirical study.

## 1. Introduction

Agents built on large language models (LLMs) perform well on tasks that are either small enough to be handled in a single reasoning step or well-specified enough that a plan can be formed in advance. As task complexity grows — many dependent steps, long feedback cycles, uncertain environment — these agents degrade. Two dominant failure modes recur across contemporary agent designs:

1. **Over-commitment to plans.** Many architectures pre-decompose a goal into a full plan before execution begins. Plans are formed under assumptions that later prove false; the agent grinds on sub-goals whose premises have been invalidated.
2. **No mechanism to question a sub-goal.** Once a sub-goal is created, it is pursued until done or abandoned by top-level control. There is no structured way for the executing unit to challenge the legitimacy of the sub-goal itself — that is, to ask "should I still be doing this?" as opposed to "how do I do this?"

These failures are, in human terms, failures of reflection on objectives rather than on methods. They are the difference between single-loop learning (detecting and correcting error relative to a goal) and double-loop learning (detecting and correcting error in the goal itself) (Argyris & Schön, 1978, 1996).

This paper proposes a framework for **steerable agent orchestration** that addresses both failure modes directly. The framework has four design pillars:

- **P1 — One layer at a time.** A complex goal is decomposed into sub-goals strictly one level per planning cycle. Deeper decomposition is deferred and evaluated as part of the sub-goal's own achieving activities (as-needed decomposition, consistent with ADaPT: Prasad et al., 2024).
- **P2 — Upward legitimacy feedback.** As part of achieving a sub-goal, the executing unit may challenge the legitimacy of the sub-goal itself. If execution evidence indicates the sub-goal's assumptions are violated, it issues feedback upward, triggering recalibration of the parent goal and its siblings.
- **P3 — The goal as an atomic unit.** Each goal and its achieving activities form a single unit — a specification, its context, its execution state, and a feedback interface to its parent. This mirrors the options framework (Sutton, Precup & Singh, 1999) and HTN task-plus-methods structure (Erol, Hendler & Nau, 1994).
- **P4 — Human as judgment, not gate.** A human engages at structured points to inject judgment, taste, and domain knowledge that prunes the search space — not to approve routine actions.

The framework is deliberately conservative: every reasoning component is existing LLM machinery (decomposition prompts, reflection loops, tool use). The novel contribution, if any, is architectural: the combination of P1 and P2 with a role for the human defined by P4, expressed as a recursive goal unit with a legitimacy channel.

The paper proceeds as follows. Section 2 reviews prior work across the disciplines the framework draws on. Section 3 states design principles and specifies the proposed architecture. Section 4 discusses the framework's relationship to prior studies and its limitations. Section 5 concludes.

## 2. Background and Related Work

### 2.1 Expert Practice as Continuous, Non-Linear Activity

Schön (1983) describes professional practice as a reflective conversation with a situation — acting, seeing consequences, and reinterpreting the situation, in a continuous loop rather than a linear "plan then execute" sequence. Dreyfus and Dreyfus (1986) describe skill acquisition as a progression in which the expert, unlike the novice, operates without explicit rule-following, noticing subtle cues and responding flexibly. Simon (1947) noted bounded rationality: human decision-makers satisfice because they cannot consider all alternatives. Taken together, these accounts suggest that human-like competence on complex goals is a property of the *process* (assessment, action, reflection) rather than of the initial plan. Our framework's P1 (deferring decomposition) follows from this view: plans are commitments made under bounded information and must remain revisable.

### 2.2 The Challenge of Complex Goals

Complex goals are ones with many interdependent parts, long feedback cycles, and uncertain environments — properties that make them wicked in Rittel and Webber's (1973) sense: every attempt to formulate a solution changes the understanding of the problem. Cynefin (Snowden & Boone, 2007) distinguishes simple, complicated, complex, and chaotic domains; decomposition-based planning is appropriate in the complicated domain, but in the complex domain the relationships between cause and effect are only knowable in hindsight, so goals must be treated as hypotheses to be tested, not directives to be executed. Simon's (1962) "architecture of complexity" notes that nearly decomposable systems — hierarchical structures with semi-independent components — are how complex systems manage themselves. Our framework adopts Simon's hierarchical stance (P3) while insisting (P1, P2) that the hierarchy remain revisable.

### 2.3 The Limits of Forward Planning

Classical planning assumed complete forward knowledge; HTN (Erol, Hendler & Nau, 1994) added hierarchical task decomposition, and SHOP2 (Nau et al., 2003) demonstrated its scalability by interleaving planning with execution. But full forward planning presupposes that goals and world state are known in advance. Suchman (1987) argued against the plan-as-blueprint view of human action, proposing instead that plans function as resources situated in action. In LLM agent terms, pre-planning approaches (e.g., BabyAGI-style task lists) suffer exactly the over-commitment failure described above. The options framework (Sutton, Precup & Singh, 1999) formalizes temporally extended courses of action with termination conditions — a plan unit that can be entered and exited — which our P3 unit borrows. ADaPT (Prasad et al., 2024) showed empirically that *as-needed* decomposition (decompose a plan element only when execution calls for it) outperforms full pre-decomposition: a finding our framework adopts as a design premise.

### 2.4 Recursive Goal Decomposition

Recursion in goal decomposition is classical in HTN planning, where tasks decompose into sub-tasks until primitive (Erol, Hendler & Nau, 1994). The novelty in modern LLM settings is not recursion per se but *when* decomposition happens and *what happens when a sub-goal becomes invalid*. Reflexion (Shinn et al., 2023) provides the inner loop: an agent self-reflects on its own failures and carries verbal reinforcement forward to improve subsequent attempts. ReAct (Yao et al., 2023) interleaves reasoning and acting, giving the agent the ability to revise its plan mid-task. However, in both ReAct and Reflexion, reflection is **single-loop**: it operates on methods relative to a fixed sub-goal. Neither provides a channel for the executing unit to challenge the legitimacy of the goal itself.

### 2.5 Double-Loop Learning and Goal Revision

Argyris and Schön (1974, 1978, 1996) distinguish single-loop learning (correcting error relative to governing variables — "am I doing this right?") from double-loop learning (correcting error in the governing variables themselves — "is this the right thing to be doing?"). Effective organizations and practitioners institutionalize double-loop learning; those that cannot fall into defensive routines. Applied to agents: single-loop reflection (Reflexion-style) keeps the agent grinding on an invalid goal with better and better technique. Our P2 is the agent-side analogue of double-loop learning: a mechanism, triggered by execution evidence, to question and revise the goal itself, propagating the question upward because the sub-goal's legitimacy derives from the parent's plan. Related work in automated planning — goal-driven autonomy (GDA) — detects and explains unexpected state and revises goals accordingly; but GDA operates at a flat plan level, not a recursive goal tree with upward propagation semantics.

### 2.6 Agent Architectures That Mirror Expert Practice

Several recent systems move toward interleaved, reflective orchestration. BabyAGI-style task-list agents continuously re-prioritize a flat list of tasks; AutoGPT-style agents run autonomous loops over user-specified goals. These validate the "continuous assessment" intuition (P1's spirit) but lack hierarchy (P3) and legitimacy feedback (P2). Multi-agent frameworks (MetaGPT, AutoGen, etc.) distribute roles but inherit the single-goal, single-loop limitation per agent. Hierarchical LLM systems (Voyager-style skill libraries) add reusable skills but are aimed at skill accumulation, not goal legitimacy. To our knowledge, no existing system combines (a) as-needed recursive decomposition, (b) an upward legitimacy feedback channel, and (c) judgment-based human steering, as a single orchestration discipline.

### 2.7 Human-in-the-Loop Collaboration

Human-in-the-loop (HITL) has a long history in ML (Mosqueira-Rey et al., 2023). The relevant design question for us is not whether to include a human but *how*: what to surface, when, and at what cost. In our framework, the human is a source of judgment — domain knowledge and taste that prune the search space — engaged at structured decision points (goal priorities, search direction, legitimacy disputes), not an approval gate over routine actions. This follows from the perspective in Section 2.1: the human adds the reflective, context-sensing capability that the agent lacks, without imposing the latency of step-by-step approval.

## 3. Proposed Framework

### 3.1 Design Principles

The framework rests on five principles:

1. **Decompose as needed, not up front (P1).** Decompose exactly one level per planning cycle; defer deeper decomposition until the executing unit needs it.
2. **Model each goal as an atomic unit (P3).** A goal unit = goal specification + context + execution state + achieving activities + a feedback interface to its parent.
3. **Allow the unit to challenge its own legitimacy (P2).** During achieving activities, if evidence indicates the sub-goal's assumptions are violated, the unit issues upward feedback rather than grinding on.
4. **Propagate upward, recalibrate downward.** On legitimacy feedback, the parent re-evaluates; if the parent recalibrates, affected siblings are re-evaluated too. Feedback is always upward; recalibration cascades down.
5. **The human steers, the agent executes (P4).** Human inputs are judgment at structured points; the agent carries out achieving activities within its unit.

### 3.2 The Goal as an Atomic Unit

Formally, a goal unit $G$ is a tuple:

$$G = (\text{spec}, \text{context}, \text{state}, \text{activities}, \text{feedback})$$

where `spec` is the goal specification, `context` is the environment and constraints, `state` is the execution state (one of *pending, active, done, abandoned*), `activities` is the procedure for achieving the goal (which may itself perform decomposition or invoke tools), and `feedback` is the interface to the parent — used to report success, failure, or a legitimacy challenge.

A top-level goal is the root of a goal tree. Each non-leaf goal, when active, may decompose into child goals; the tree is built one level per cycle (P1). The `focus` of execution is always exactly one active leaf.

### 3.3 Decomposition Protocol

When an active goal cannot be achieved directly (no primitive action is available), it is decomposed into child goals, each with its own spec, context, and state. Decomposition is *as-needed*: a goal is decomposed only when its achieving activities require it, and only one level at a time. Deeper structure is not anticipated; it is discovered as execution proceeds. Each child inherits a legitimacy premise from its parent: the child is assumed legitimate because the parent's plan says so. This premise is what P2 makes challengeable.

### 3.4 Legitimacy Challenging and Parent-Goal Calibration

The central mechanism. As a unit executes, it gathers evidence. If the evidence indicates that the sub-goal's *premises* are violated — the world has changed, the parent's intent was misread, or the goal is no longer worth its cost — the unit may issue a **legitimacy challenge** upward via its feedback interface, rather than continuing to grind.

The parent, on receiving a challenge, has three options:

- **Confirm.** The parent reasserts the sub-goal (possibly with revised context); execution resumes.
- **Recalibrate.** The parent revises the sub-goal, or the parent's own plan; the parent may decompose a replacement.
- **Abandon.** The parent acknowledges the goal is no longer legitimate and marks it abandoned; the parent may itself challenge its own legitimacy (recursion).

Because a recalibration changes the parent's commitments, affected siblings are re-evaluated (upward propagation, downward cascade). This is double-loop learning applied to a goal tree.

### 3.5 Human Steering

The human operates at the *judgment* layer. Structured interaction points include: setting and re-ordering top-level priorities; resolving legitimacy disputes where the agent's evidence is ambiguous (the agent argues, the human decides); and steering search direction when the goal tree has multiple plausible decompositions. Human attention is bounded by design: the agent requests input only at structured, high-leverage points. This is P4: human as judgment source, not approval gate.

## 4. Discussion

### 4.1 Alignment with Prior Studies

The framework is a synthesis of established ideas. Its recursive structure echoes Simon (1962) and HTN planning; its as-needed decomposition adopts ADaPT's (Prasad et al., 2024) empirical finding; its unit structure borrows from the options framework (Sutton, Precup & Singh, 1999); its inner loop is ReAct/Reflexion (Yao et al., 2023; Shinn et al., 2023); its legitimacy mechanism is double-loop learning (Argyris & Schön, 1978, 1996) and GDA applied recursively; its human role follows HITL-ML (Mosqueira-Rey et al., 2023) and the expert-practice literature (Schön, 1983). The contribution is not a new technique but a new *arrangement*: the combination of P1, P2, P3, P4 into one orchestration discipline.

### 4.2 The Role of Human Judgment

Why should a human be in the loop at all, given capable agents? Because judgment — deciding which goals are worth pursuing, when a goal's premises have failed, and which direction the search should take — is precisely what the practice literature says distinguishes expert performance from rule-following (Schön, 1983; Dreyfus & Dreyfus, 1986). Bounded rationality (Simon, 1947) explains why a satisficing human who prunes a vast search space is valuable even when imperfect. The design challenge, handled in Section 3.5, is to capture this value without reintroducing the bottleneck that fully autonomous designs seek to avoid.

### 4.3 Limitations and Open Questions

The framework is a design proposal, not an empirical result. Key open questions:

- How to define and measure "one level of decomposition" objectively (P1 is underspecified until this is settled).
- What confidence thresholds should gate (a) deferring deeper decomposition and (b) issuing a legitimacy challenge — too low and the system oscillates; too high and the mechanism is inert.
- Whether a legitimacy challenge should cascade to siblings always, or only when the parent actually recalibrates.
- Whether some goals should be forbidden from self-challenge (e.g., hard user constraints), to prevent the agent from talking itself out of user requirements.
- When to cache reusable sub-goal strategies (skill libraries) versus re-decomposing.

## 5. Conclusion

This paper proposed a framework for steerable agent orchestration based on recursive goal decomposition with upward legitimacy feedback. Its four pillars — one-layer-at-a-time decomposition (P1), sub-goal legitimacy challenging with upward propagation (P2), the goal as an atomic unit (P3), and human-as-judgment steering (P4) — directly target the two dominant failure modes of LLM agents on complex goals: over-commitment to plans and the inability to question a sub-goal. The framework is grounded in prior work across expert practice, planning, and organizational learning, and its components are all existing LLM machinery; the candidate contribution is architectural. Empirical evaluation of the framework — particularly whether P2's upward legitimacy feedback reduces wasted work relative to fixed-goal baselines, and whether P4's bounded human steering improves outcomes — is the natural next step.

## References

- Argyris, C., & Schön, D. A. (1974). *Theory in Practice: Increasing Professional Effectiveness*. Jossey-Bass.
- Argyris, C., & Schön, D. A. (1978). *Organizational Learning: A Theory of Action Perspective*. Addison-Wesley.
- Argyris, C., & Schön, D. A. (1996). *Organizational Learning II: Theory, Method, and Practice*. Addison-Wesley.
- Ashby, W. R. (1956). *An Introduction to Cybernetics*. Chapman & Hall.
- Boyd, J. (1995). *The Essence of Winning and Losing* (unpublished briefings).
- Dewey, J. (1933). *How We Think*. D. C. Heath.
- Dreyfus, H. L., & Dreyfus, S. E. (1986). *Mind over Machine: The Power of Human Intuition and Expertise in the Era of the Computer*. Free Press.
- Erol, K., Hendler, J., & Nau, D. S. (1994). UMCP: A sound and complete procedure for hierarchical task-network planning. In *Proceedings of the 2nd International Conference on AI Planning Systems (AIPS)*.
- Mosqueira-Rey, E., Hernández-Pereira, E., Alonso-Ríos, D., Bobes-Bascarán, J., & Fernández-Leal, Á. (2023). Human-in-the-loop machine learning: A state of the art. *Artificial Intelligence Review*, 56, 3005–3054.
- Nau, D. S., Au, T.-C., Ilghami, O., Kuter, U., Murdock, J. W., Wu, D., & Yaman, F. (2003). SHOP2: An HTN planning system. *Journal of Artificial Intelligence Research*, 20, 379–404.
- Polk, T. A. (1999). *Computational Models of Skill Acquisition*. Oxford University Press.
- Prasad, A., Koller, A., Hartmann, M., Clark, P., Sabharwal, A., Bansal, M., & Khot, T. (2024). ADaPT: As-needed decomposition and planning with language models. *Proceedings of the 2024 Conference of the North American Chapter of the Association for Computational Linguistics (NAACL)*.
- Rittel, H. W. J., & Webber, M. M. (1973). Dilemmas in a general theory of planning. *Policy Sciences*, 4(2), 155–169.
- Schön, D. A. (1983). *The Reflective Practitioner: How Professionals Think in Action*. Basic Books.
- Shinn, N., Cassano, F., Gopinath, A., Narasimhan, K., & Yao, S. (2023). Reflexion: Language agents with verbal reinforcement learning. *Advances in Neural Information Processing Systems (NeurIPS)*.
- Simon, H. A. (1947). *Administrative Behavior*. Macmillan.
- Simon, H. A. (1955). A behavioral model of rational choice. *Quarterly Journal of Economics*, 69(1), 99–118.
- Simon, H. A. (1962). The architecture of complexity. *Proceedings of the American Philosophical Society*, 106(6), 467–482.
- Simon, H. A. (1979). *Models of Thought*. Yale University Press.
- Snowden, D. J., & Boone, M. E. (2007). A leader's framework for decision making. *Harvard Business Review*, 85(11), 68–76.
- Suchman, L. A. (1987). *Plans and Situated Actions: The Problem of Human-Machine Communication*. Cambridge University Press.
- Sutton, R. S., Precup, D., & Singh, S. (1999). Between MDPs and semi-MDPs: A framework for temporal abstraction in reinforcement learning. *Artificial Intelligence*, 112(1–2), 181–211.
- Wiener, N. (1948). *Cybernetics, or Control and Communication in the Animal and the Machine*. MIT Press.
- Yao, S., Zhao, J., Yu, D., Du, N., Shafran, I., Narasimhan, K., & Cao, Y. (2023). ReAct: Synergizing reasoning and acting in language models. *International Conference on Learning Representations (ICLR)*.
