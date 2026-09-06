# Paper II — An Epistemic Operating System: Engineering Architecture for a Viability-Constrained Agent

**Status:** Engineering blueprint, staged and gated by the falsification criteria in Paper I. No component below is included unless it corresponds to a specific formal object in Paper I or a specific, cited precedent in the systems literature. Where a component is speculative, it is placed in the deferred-hardware appendix rather than the core architecture, and marked as such.

---

## Abstract

Paper I establishes what must be true, mathematically, for a viability-constrained, epistemically-driven agent to prefer informative action over inaction, and identifies exactly which of its claims are proven versus conjectured. This paper asks the different question: what computational architecture would actually run that agent, at what cost, and in what order should its pieces be built and tested so that a failure at any stage is informative rather than ambiguous? We define an "epistemic operating system" whose scheduled resources are not files and processes but belief states, hypotheses, memory tiers, and candidate experiments — each service in the OS corresponds one-to-one with a formal object from Paper I, and each is cited against its true precedent in cognitive architecture, model-based reinforcement learning, and systems research rather than presented as novel where it is not. We give a concrete complexity analysis showing why naive policy search is intractable and why hierarchical temporal abstraction is a load-bearing engineering requirement, not an optional refinement. We specify a staged build order — an experimental ladder — in which each stage is gated by a specific falsification test from Paper I §10, and no stage is built before the previous one has produced a measurable result. Hardware is treated identically: every accelerator beyond a commodity CPU/GPU baseline is deferred to a profiling-triggered appendix, following directly from the arithmetic-intensity and Amdahl's-law analysis of where GPU compute is and is not matched to this workload.

---

## 1. Scope and Relationship to Paper I

Every section heading below states, in parentheses, which Paper I object it implements. This is not a formatting nicety — it is the mechanism that prevents this document from drifting into the failure mode identified repeatedly across this research program: an unspecified transition function $\Phi$ or $\Psi$ standing in for work that has not been done. If a system component below cannot be traced to a specific $b_t$, $\Theta$, $IG$, $CIG$, $LP$, or $\Pi_{\text{safe}}$ from Paper I, it does not belong in the core architecture, and is either cut or moved to an explicitly-labeled speculative appendix.

---

## 2. Core State (implements Paper I Definitions 1–6)

$$S_t = (b_t, K_t, \pi_t, Q_t)$$

where $b_t = P(s_t, \theta, M \mid h_t)$ is the full Bayesian belief (Paper I §2), $K_t$ is the current viability kernel estimate and admissible region (Paper I §3), $\pi_t$ is the currently deployed policy, and $Q_t$ is the computational-resource state (available compute, memory, energy budget — a state variable in its own right, since Paper I's $J(\pi)$ explicitly charges for $C_{\text{compute}}(\pi)$). This is deliberately smaller than the eight-way decomposition in earlier working drafts ($R_t, M_t, W_t, H_t, V_t, Q_t, P_t, X_t$): every one of those eight objects is recoverable as a projection of $b_t$ (the world model is the state-and-mechanism marginal of $b_t$; the hypothesis space is the $M$-marginal; the self-model is the sub-belief over $Q_t$-relevant variables), and collapsing them avoids implying they are independent state variables requiring independent update rules when in fact they are one joint posterior being queried in different ways.

---

## 3. The Epistemic Operating System

A conventional OS schedules CPU time and memory pages among processes. This OS schedules **belief-update operations and candidate interventions** among a fixed compute and energy budget. Each service below is defined by the Paper I object it maintains or queries, not by an analogy to biology.

### 3.1 Belief Service (implements $b_t$, Paper I §2)

Maintains the joint posterior over $(s_t, \theta, M)$. **Precedent:** this is a Bayes-adaptive POMDP solver (Duff, 2002; Ghavamzadeh et al., 2015) — not a new component. The engineering question is exclusively representational: exact posterior maintenance is intractable outside toy state spaces, so the belief service must use one of (a) a particle filter / sequential Monte Carlo approximation, (b) a variational approximation (amortized via a learned encoder, as in Hafner et al.'s 2019 "PlaNet" and 2020 "Dreamer" latent world models, which are the closest existing working systems to this service's required function), or (c) an ensemble of point-estimate models standing in for a discrete approximation of $P(M \mid h_t)$ (Chua et al., 2018, "PETS," is the standard reference implementation of the ensemble approach for model-based RL). **Recommendation for Stage 0 (§9): ensemble approximation.** It is the cheapest to implement correctly and the easiest to debug, at the cost of a coarser posterior than a full variational model would give — an acceptable trade for a first validation pass.

### 3.2 Viability Service (implements Definitions 1–2, Proposition 5, Open Problem 3)

Computes $K_t$ and evaluates $V_H(b_t, \pi) \ge 1-\epsilon$ for candidate policies before they are permitted onto the scheduler. **This service has veto power over every other service in the OS — it is not one bidder in a market of objectives, it is a gate**, which is the direct engineering consequence of Assumption 1 in Paper I (viability as a hard constraint on $\Pi_{\text{safe}}$, not an additive reward term). Given Proposition 5's compounding-risk result, this service must not report a single static $\epsilon$ as a lifetime guarantee; it must expose a *remaining risk budget* that decays with elapsed decisions and can trigger a mandatory conservative mode (fall back to $a_0$-adjacent low-risk actions) when the budget is nearly exhausted, pending Open Problem 3's unresolved question of where the replenishment or re-certification of that budget should come from.

### 3.3 Curiosity Service (implements $IG$, $CIG$, $LP$ — Definitions 3–5)

Computes the three epistemic terms for each candidate action, using the belief service's current posterior. This service should be structured as three independently swappable sub-modules precisely because Paper I's falsification criterion F1 requires running $IG$-only and $CIG$-only conditions head to head — if the service does not cleanly separate these, the ablation required to test F1 cannot be run without rewriting the system.

### 3.4 Value / Goal Service (implements $G(\pi)$ and Theorem 3's corollary)

Maintains whatever externally- or developer-specified task value exists, if any. Theorem 3 proves this cannot be empty and simultaneously produce non-null behavior; the engineering consequence is that this service must always report at least one nonzero-weighted term into $J(\pi)$ even in the "no external task" condition, and that term should be logged and disclosed as an explicit design choice, not treated as if it disappears when set to a constant.

### 3.5 Scheduler / Computational Attention (implements $\arg\max_\pi J(\pi)$ subject to $\Pi_{\text{safe}}$, §5 below for complexity)

Selects which candidate actions to evaluate at all, given that full enumeration is intractable (§5). Precedent: this is functionally a meta-level bandit or Monte Carlo Tree Search resource allocator (MCTS as used in MuZero, Schrittwieser et al., 2020, is the closest large-scale precedent for planning under a learned model with a resource budget).

### 3.6 Memory Service (implements the empirical estimate of $b_t$'s sufficient statistics over time — §7)

Distinct from the belief service: the belief service holds the *current* posterior; the memory service holds the evidence trace needed to detect that the posterior should be revised (Grünwald & van Ommen misspecification risk from Paper I §5) and to support the consolidation process described in §7.

### 3.7 Safety / Corrigibility Layer (§6)

Sits between the scheduler and the actuator interface; the only service permitted to reject an already-selected action outright, distinct from the viability service's role of shaping which actions are ever candidates.

---

## 4. The Core Loop

```
STATE: b_t (belief), K_t (viability kernel), Q_t (resources)

loop:
    o_t ← observe(environment)
    b_t ← belief_service.update(b_t, o_t, a_{t-1})          # Bayes update, §3.1

    K_t ← viability_service.update(K_t, Q_t)                 # §3.2
    Π_safe ← viability_service.feasible_actions(b_t, K_t)    # hard filter, precedes scoring

    if Π_safe is empty or near-exhausted:
        a_t ← conservative_fallback_policy(b_t)               # Open Problem 3 mitigation
    else:
        candidates ← scheduler.propose(Π_safe, Q_t)           # bounded by compute budget, §5
        for a in candidates:
            IG, CIG, LP ← curiosity_service.evaluate(a, b_t)  # §3.3, independently swappable
            G ← value_service.evaluate(a, b_t)                # §3.4
            J[a] ← G + λ_I·IG + λ_C·CIG + λ_L·LP − λ_R·R(a) − λ_K·cost(a)
        a_t ← safety_layer.filter(argmax_a J[a])              # §3.7, can veto

    execute(a_t)
    o_{t+1} ← observe(environment)
    error ← compare(predicted(a_t), o_{t+1})
    memory_service.log(b_t, a_t, o_{t+1}, error)              # §3.6, feeds §7 consolidation
    Q_t ← Q_t − cost(a_t)
```

Every line above is traceable to a named object in Paper I or an explicitly cited precedent; there is no line whose function is "and then the system figures it out."

---

## 5. Complexity Analysis and the Case for Hierarchy

Naive evaluation of the loop above, enumerating actions to horizon $H$ over $|\mathcal{A}|$ actions and $|M|$ candidate causal models, costs

$$O(|M| \cdot |\mathcal{A}|^H)$$

per decision. This is not a hypothetical concern to be waved past — it is the actual dominant cost, and for any nontrivial $|\mathcal{A}|$ and $H$ it is intractable within any realistic per-decision compute budget $Q_t$. Two independent, well-precedented mitigations are required, not optional:

**Temporal abstraction.** Actions are replaced by options — temporally extended sub-policies with their own initiation and termination conditions (Sutton, Precup & Singh, 1999, "Between MDPs and Semi-MDPs: A Framework for Temporal Abstraction in Reinforcement Learning" — the correct, original citation for this mechanism). This reduces the effective horizon $H$ by replacing single-step actions with multi-step skills, at the cost of requiring the option set itself to be learned or engineered, which is a separate, nontrivial problem this paper does not solve and flags as **Open Problem 4**.

**Model-based rollout rather than exhaustive search.** Rather than enumerating $|\mathcal{A}|^H$ sequences explicitly, MCTS with a learned value/policy prior (Schrittwieser et al., 2020) samples a small fraction of the tree guided by the current $J$-estimates, trading completeness for tractability. This is the mechanism, not hierarchical abstraction in the representational sense discussed in the theory-of-intelligence line of this research program (Tishby & Zaslavsky's Information Bottleneck) — the two are related but distinct, and should not be conflated: one compresses the *action* space, the other compresses the *state representation*. Both are needed, and neither is optional at any nontrivial scale.

---

## 6. Safety and Corrigibility Architecture (implements Paper I §7's wireheading and paralysis rows)

Two specific engineering requirements follow directly from Paper I's failure-mode table, and are stated as hard requirements, not design preferences:

**Requirement S1 (wireheading prevention).** The channel that computes $J(\pi)$ — belief service, curiosity service, value service — must be causally downstream of, and structurally non-writable by, any action in $\mathcal{A}$. This is not achievable by a software permission flag alone; it requires the evaluation channel to run on a separate, more restricted execution context than the action-selection and actuation path, following the general architectural principle (not the specific mechanism) of Orseau & Armstrong's (2016) "Safely Interruptible Agents," and the broader treatment of reward-tampering as a distinct failure class in Amodei et al. (2016), "Concrete Problems in AI Safety."

**Requirement S2 (corrigibility under self-modification).** Any candidate change to the belief service's model class, the curiosity service's weighting, or the scheduler's search strategy must pass through the sandboxed proposal → validate → benchmark → regression-test → approve pipeline before deployment (this specific pipeline is an engineering control, not a theorem; it is included here as a requirement precisely because Theorem 3 shows the system's values are not self-justifying — an unconstrained self-modification process has no internal check that would prevent it from modifying $\lambda_I, \lambda_C, \lambda_L$ toward degenerate values, since nothing in the mathematics of Paper I forbids that).

---

## 7. Memory Architecture (implements the evidence trace behind $b_t$'s updates over time)

**Precedent, stated honestly:** the working/episodic/semantic split below is not an arbitrary engineering choice — it is the direct systems analogue of Complementary Learning Systems theory (McClelland, McNaughton & O'Reilly, 1995), which argues, from actual neuroscience evidence (hippocampal rapid encoding versus neocortical slow consolidation), that a single fast-learning system cannot both retain specific recent episodes *and* extract stable general structure without one destructively interfering with the other — the same stability–plasticity tension named in the continual-learning literature (De Lange et al., 2022; Kirkpatrick et al., 2017, "Elastic Weight Consolidation," for a concrete mechanism addressing exactly this interference).

| Tier | Formal role | Consolidation trigger |
|---|---|---|
| Working | Active $b_t$ and in-flight candidate evaluations | N/A — volatile |
| Episodic | Raw $(b_t, a_t, o_{t+1}, \text{error})$ traces, per Grünwald & van Ommen misspecification detection | Written on every step; read when belief-service posterior needs re-derivation from scratch |
| Semantic | Consolidated mechanism estimates — effectively a compressed cache of the $M$-marginal of past $b_t$'s that have stabilized | Promoted when a mechanism's posterior confidence exceeds a threshold and remains stable across a window of new evidence (directly parallel to the consolidation criterion already used in this research program's earlier causal-engine work) |

---

## 8. World Model / Causal Engine Service

The belief service (§3.1) maintains the posterior; this service supplies the **mechanism library** it draws on — the actual candidate structural forms $M$ can take. **Precedent for the causal half:** the PC algorithm (Spirtes & Glymour) for skeleton discovery from passive data, combined with Hauser & Bühlmann's (2012) and Hyttinen, Eberhardt & Hoyer's (2013) intervention-selection results for the active, CIG-driven half. **Precedent for the latent-dynamics half:** recurrent state-space models as in Hafner et al.'s Dreamer/PlaNet line are the closest existing working implementations of a learned $P(z_{t+1} \mid z_t, a_t)$ suitable as the non-causal component of $M$ before causal structure is imposed on top of it.

---

## 9. Novelty Ledger

| System component | True precedent | What (if anything) is new here |
|---|---|---|
| Belief service | Bayes-adaptive POMDPs (Duff, 2002); Dreamer/PlaNet (Hafner et al.) | Nothing in the mechanism; new only in being gated by a hard viability constraint (§3.2) rather than run unconstrained |
| Viability service | Viability theory (Aubin, 1991); constrained MDPs (Altman, 1999); safe RL survey (García & Fernández, 2015) | Nothing — direct application |
| Curiosity service ($IG$) | Lindley (1956); Houlsby et al. (2011) | Nothing |
| Curiosity service ($CIG$) | Hauser & Bühlmann (2012); Hyttinen, Eberhardt & Hoyer (2013) | Embedding in a persistent, non-episodic agent loop rather than a one-shot experimental-design problem — a real but modest contribution |
| Curiosity service ($LP$) | Schmidhuber (1991, 2010) | Nothing |
| Scheduler | MCTS with learned priors (Schrittwieser et al., 2020); options framework (Sutton, Precup & Singh, 1999) | Nothing — direct application to this state space |
| Memory tiering | Complementary Learning Systems (McClelland, McNaughton & O'Reilly, 1995); EWC (Kirkpatrick et al., 2017) | Nothing |
| Safety layer | Orseau & Armstrong (2016); Amodei et al. (2016) | Nothing in mechanism — the requirement that it be structurally downstream of the reward channel is a direct, undisputed consequence of Theorem 3, not a new safety idea |
| **The coupling of all of the above under one hard viability gate with three separable epistemic terms** | No single existing system combines exactly this set | **This is the actual candidate contribution of the entire program**, and it is untested — see §13 |

This table should be read as the honest answer to "what is new here": almost nothing at the component level, and one specific, testable claim at the system level.

---

## 10. Software Stack (staged, not maximal)

**Stage 0 requires:** Python, NumPy/JAX for the ensemble belief service, a hand-written toy gridworld (no simulation engine needed at this scale), and a plain logging/experiment-tracking setup sufficient to reproduce Paper I §10's four-condition test. Nothing else. Introducing PyTorch, a distributed scheduler, vector databases, or graph stores before Stage 0 has produced a result is scope creep against the "smallest thing that could kill it" principle this whole program has converged on.

**Stage 1+ (only after Stage 0 passes):** PyTorch or JAX for a learned ensemble or variational belief service at larger state-space scale; a proper experiment database (even SQLite is sufficient at this scale) to log the Experiment record structure needed for the falsification program in Paper I §10.

---

## 11. Hardware Roadmap (profiling-gated, per the Roofline/Amdahl analysis established earlier in this research program)

**Baseline (Stage 0–1): single CPU, optionally one commodity GPU.** The belief service's ensemble updates and the curiosity service's mutual-information estimates are the dominant costs at this scale, and both are dense numerical operations well matched to CPU/GPU arithmetic intensity — there is no engineering justification for anything more at this stage.

**Deferred appendix — not part of the core architecture until triggered by measured bottlenecks:** FPGA or compute-in-memory acceleration for the causal engine's discrete graph search (justified only if profiling shows the PC-algorithm / intervention-selection step, which is branch-heavy and low-arithmetic-intensity exactly as characterized by the Roofline model in this program's earlier physics-of-intelligence work, is the measured bottleneck — not before). Neuromorphic or photonic acceleration is not justified at any stage currently reachable by this program's roadmap and is omitted entirely rather than listed as a "future possibility," since listing unjustified hardware options is precisely the kind of scaffolding that looks like engineering rigor while adding none.

---

## 12. The Experimental Ladder (each stage gated by a named Paper I falsification criterion)

**Stage 0 — the four-condition test.** Null baseline, $G$-only, $G{+}IG$, $G{+}CIG$, in the minimal toy environment already specified (grid, energy, hidden red/blue causal rule, no reward leakage). **Gate to proceed to Stage 1:** F1 and F2 (Paper I §10) must both fail to trigger — i.e., $CIG$ must measurably outperform $IG$ at causal discrimination, and the exploration preference from Theorem 2 must survive a realistic, nonzero cost gap $\Delta$. If either F1 or F2 triggers, the correct response is to revise Paper I's objective form (Proposition 4's convexity issue is the first place to look), not to proceed to a larger environment hoping the problem resolves at scale.

**Stage 1 — richer causal structure.** Multiple interacting hidden variables, requiring the PC-algorithm/intervention-selection machinery (§8) rather than the two-hypothesis toy case. Gate: causal discovery accuracy against the known ground-truth structure, compared to a passive-observation-only baseline.

**Stage 2 — introduce the viability constraint under genuine resource scarcity**, testing Proposition 5's compounding-risk concern directly by running the agent for a long horizon and measuring cumulative failure probability against the theoretical bound.

**Stage 3 and beyond are not specified here.** Per this program's own stated principle, specifying Stage 3's architecture before Stage 0's result exists would be designing memory budgets and scheduler complexity for a mechanism that might not have survived its first test.

---

## 13. What This Blueprint Is Not

This is not a specification for AGI. It is a specification for the smallest system capable of testing one falsifiable claim from Paper I. Every component beyond that minimal system — the memory hierarchy, the hardware roadmap, the corrigibility pipeline — is included because it will be needed *if* Stage 0 succeeds, not because its inclusion here constitutes evidence that it will.

---

## References

(Shared references with Paper I are not repeated; the following are additional to this document.)

Amodei, D. et al. (2016). Concrete problems in AI safety. *arXiv:1606.06565.*
Chua, K. et al. (2018). Deep reinforcement learning in a handful of trials using probabilistic dynamics models. *NeurIPS.*
De Lange, M. et al. (2022). A continual learning survey: defying forgetting in classification tasks. *IEEE TPAMI.*
Hafner, D. et al. (2019). Learning latent dynamics for planning from pixels (PlaNet). *ICML.*
Hafner, D. et al. (2020). Dream to control: learning behaviors by latent imagination (Dreamer). *ICLR.*
Kirkpatrick, J. et al. (2017). Overcoming catastrophic forgetting in neural networks. *PNAS.*
McClelland, J., McNaughton, B. & O'Reilly, R. (1995). Why there are complementary learning systems in the hippocampus and neocortex. *Psychological Review.*
Orseau, L. & Armstrong, S. (2016). Safely interruptible agents. *UAI.*
Schrittwieser, J. et al. (2020). Mastering Atari, Go, chess and shogi by planning with a learned model (MuZero). *Nature.*
Sutton, R., Precup, D. & Singh, S. (1999). Between MDPs and semi-MDPs: a framework for temporal abstraction in reinforcement learning. *Artificial Intelligence.*
