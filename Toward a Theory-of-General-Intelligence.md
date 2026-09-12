# Paper I — Toward a Theory of General Intelligence: Adaptive Model-Building, Prediction, Learning, and Self-Revision

**Status:** Research hypothesis and falsifiable experimental program.

**Revision log — ruthless pass, what was cut and why:**
- **Cut** the "measurable interpretation of intuition" section entirely. It renamed the adaptive-computation hypothesis (H4) in different words without adding a distinct testable claim. Decoration, not content.
- **Cut** "the deeper conjecture" and "research consequence" sections entirely. Both were unfalsifiable as written ("intelligence may be fundamentally the ability to...") and added no claim beyond the hypothesis matrix already in §17. A grand unifying sentence that cannot fail is not a scientific contribution; it is a mission statement, and mission statements do not belong in a document whose whole purpose is falsifiability.
- **Downgraded** the claim that interaction effects "may be more scientifically important than any individual module" — this was the least-grounded superlative in the paper. It is now a hypothesis with a stated failure condition, not an assertion.
- **Added** an explicit scope limit on "generality" (§4) forced by the No Free Lunch theorem, which this research program already relies on elsewhere and cannot selectively ignore here.
- **Added** a primary/exploratory split to the hypothesis matrix (§17) and an explicit dependency graph (§18), because treating eight hypotheses as independent when several structurally presuppose others passing first was a real defect, not a stylistic choice.
- **Removed** the escape-hatch phrase "sufficiently demanding benchmarks" from the falsification criteria (§19) and replaced it with a pre-registration requirement, because as written the criteria could never actually be triggered — any disappointing result could be attributed to the benchmark being insufficiently demanding, after the fact.

What survives this pass is smaller than the original draft. That is the intended outcome, not a side effect.

---

## 1. The problem

AI has advanced by discovering computational structures that make particular forms of learning tractable: deep networks for high-dimensional function approximation, the Transformer for content-dependent sequence interaction (Vaswani et al., 2017), reinforcement learning for feedback-driven action selection, learned world models for planning in compact latent space rather than raw observation space (Hafner, Pasukonis, Ba & Lillicrap, 2025, *Nature* 640:647–653 — DreamerV3, which solved over 150 tasks across 8 domains with one fixed hyperparameter configuration), selective state-space models as an alternative to attention (Gu & Dao, 2023, arXiv:2312.00752; COLM 2024), and Vector Symbolic Architectures / Hyperdimensional Computing as a materially different formalism for compositional distributed representation (Kleyko, Rachkovskij, Osipov & Rahimi, 2022/2023, *ACM Computing Surveys* 55(6):130 and 55(9):175).

None of this establishes a complete account of intelligence. The danger is premature architectural commitment: "Transformers are insufficient, therefore architecture X is the answer" collapses a search space into a preference before the space has been searched. The question this program asks instead: which computational capabilities become *necessary* when a system is placed in a world that is novel, partially observable, stochastic, dynamic, embodied, long-horizon, resource-constrained, and capable of invalidating the system's own assumptions?

---

## 2. Intelligence as an adaptive process

An agent in environment $E$ does not observe true state $S_t$ directly; it receives $O_t \sim P(O_t \mid S_t)$, and $S_{t+1} \sim P(S_{t+1} \mid S_t, A_t)$. This creates three simultaneous problems — infer, predict, control — and, in a genuinely open environment, a fourth: **revise the model when inference or prediction fails.**

---

## 3. A candidate computational loop

$$\text{Observe} \to \text{Represent} \to \text{Compress} \to \text{Remember} \to \text{Predict} \to \text{Estimate uncertainty} \to \text{Allocate computation} \to \text{Act} \to \text{Observe consequences} \to \text{Revise}$$

Three levels are distinguished and tested separately, because conflating them is the most common way this kind of program overclaims:

- **Functional hypothesis** — a capability exists if the system performs the relevant computational function, regardless of implementation.
- **Architectural hypothesis** — a particular organization is an efficient way of implementing that function.
- **Hardware hypothesis** — a particular physical substrate performs the underlying operations efficiently.

A functional result ("uncertainty-driven action helps") does not license an architectural conclusion, and an architectural conclusion does not license a hardware conclusion, without independent evidence at each level.

---

## 4. What "general" means, operationally  and what it cannot mean

Let a task distribution be $\mathcal{T} = \{T_1, \ldots, T_n\}$. A system is more general when performance holds under changes to task structure, environment geometry, object identity, physical dynamics, available information, sensor configuration, temporal horizon, objectives, interaction rules, and novel combinations of known components — i.e., $P_{\text{environment}} \neq P_{\text{training}}$, not merely a held-out split from the same distribution.

**This must be stated as a bounded claim, not an absolute one.** The No Free Lunch theorem (Wolpert & Macready, 1997) proves there is no ranking of agents by generality that is independent of an assumed distribution over possible environments — averaged uniformly over all possible environments, no agent outperforms any other. Every generality claim in this paper is therefore *generality conditional on the tested shift axes*, not generality in the unconditioned sense the word suggests in ordinary use. A system that generalizes across all seven named axes above has demonstrated something real and useful; it has not demonstrated general intelligence in any distribution-free sense, because no experiment can.

---

## 5. The central hypothesis

> General intelligence may emerge more readily from systems that continuously construct, maintain, test, and revise internal models of their environment and themselves, while adaptively allocating representation, memory, sensing, and computation according to uncertainty, objectives, environmental change, and expected value.

Deliberately weaker than "world models produce AGI," deliberately broader than "use a Transformer plus memory."

---

## 6. Candidate dimensions of intelligence

$$\mathcal{I} = \{P, R, W, M, L, C, U, A, G, V, S, D\}$$

| Symbol | Capability |
|---|---|
| $P$ | Perception | $R$ | Representation | $W$ | World modeling | $M$ | Memory |
| $L$ | Learning | $C$ | Adaptive computation | $U$ | Uncertainty estimation | $A$ | Action and control |
| $G$ | Goal formation | $V$ | Abstraction | $S$ | Self-modeling | $D$ | Diagnosis and model revision |

The question is not whether all twelve exist as explicit modules — it is whether generality requires functional equivalents of some combination of them, and which.

---

## 7. Perception as an active process

$$A_t^{\text{sense}} = \arg\max_a \left[ \mathbb{E}[\mathcal{U} \mid a] - \lambda C(a) \right]$$

Active-perception research already studies information-gain-driven sensing policies in both simulation and hardware. The hypothesis: a sufficiently general agent may need to actively manipulate what it observes, not passively consume a fixed stream.

---

## 8. Representation as a constrained optimization

$$\min_\Phi C(Z_t) \quad \text{s.t.} \quad \mathcal{E}_{\text{prediction}} \le \epsilon_p, \quad \mathcal{E}_{\text{control}} \le \epsilon_c, \quad \mathcal{E}_{\text{generalization}} \le \epsilon_g$$

A direct application of rate–distortion / Information Bottleneck framing (Tishby & Zaslavsky, 2015) to an online agent — this turns representation quality into a measurable object.

---

## 9. Prediction and intervention

$P(S_{t+1} \mid S_t)$ vs. $P(S_{t+1} \mid S_t, A_t)$, extended to $P(S_{t+\tau} \mid S_t, A_{t:t+\tau-1})$. Verified precedent:

- **V-JEPA 2** (Assran et al., FAIR at Meta/Mila, arXiv:2506.09985, June 2025): self-supervised video pretraining on 1M+ hours, action-conditioned post-training on under 62 hours of robot video, zero-shot pick-and-place on physical Franka arms with no task-specific reward.
- **DreamerV3** (Hafner et al., *Nature* 640:647–653, 2025): compact latent world model, planning through imagined rollouts, 150+ tasks, fixed hyperparameters.
- **Genie 2** (Parker-Holder et al., Google DeepMind technical report, Dec. 2024) — a technical announcement, **not peer-reviewed**; weighted accordingly.

Open question this program actually tests: does an agent need *different* predictive models at different temporal/task scales simultaneously, rather than one model for all horizons? None of the above systems answers this — each targets one operating regime.

---

## 10. Uncertainty as a control signal

$C_t = \pi_C(Z_t, U_t, Q_t, \Delta Z_t, R_t)$ — spend more computation exactly when being wrong is consequential or the state is poorly known.

---

## 11. Adaptive computation

Test-time compute scaling shows additional inference-time computation can outperform more parameters on some reasoning tasks, with the optimal allocation depending on question difficulty (Snell, Lee, Xin & Kumar, 2024, arXiv:2408.03314) — not a universal law, a task-dependent finding.

---

## 12. Memory as a selection problem

$$M_{t+1} = \text{Update}(M_t, E_t, \eta_t), \qquad r(x) = f(n(x), u(x), c(x), p(x), v(x))$$

Catastrophic forgetting is a documented, unresolved difficulty for sequential neural learning (De Lange et al., 2022, IEEE TPAMI) — memory selection responds to a real failure mode, not a hypothetical one.

---

## 13. Abstraction as measurable transfer

$H = \Psi(E_1, \ldots, E_n)$ is useful exactly when it permits shorter, more accurate, or more general prediction/control in structurally related, previously unseen situations. Operational, not honorific.

---

## 14. Causal understanding

$P(Y\mid X) \ne P(Y\mid do(X{=}x))$; active-intervention-selection machinery is treated in the companion viability-constrained-agency work (Hauser & Bühlmann, 2012; Hyttinen, Eberhardt & Hoyer, 2013). Test here: does explicit causal modeling improve adaptation to *novel mechanisms* more than correlation alone, holding data and compute fixed?

---

## 15. Goal formation

$G_{t+1} = \Gamma(G_t, Z_t, U_t)$ — hierarchical goal construction, tested independently from ordinary policy learning rather than assumed to emerge from it.

---

## 16. Self-modeling

$A_t = \pi(Z_t^{\text{world}}, Z_t^{\text{self}}, Q_t)$ — links self-modeling to active perception (§7): "I cannot determine this from here" → "move."

---

## 17. Self-revision

$\epsilon_t = O_{t+1} - \hat{O}_{t+1}$; a shallow system memorizes the exception, a stronger system diagnoses which assumption produced $\epsilon_t$ and revises: $W_{t+1} = \text{Revise}(W_t, \epsilon_t)$. This paper's interest is model revision specifically, not learning in general.

---

## 18. Interaction effects, and the dependency structure between hypotheses

For mechanisms $A, B$: $\Delta_{AB} = I(A{+}B) - I(A) - I(B) + I(\varnothing)$.

**This quantity is defined only relative to the task distribution it is computed over.** A synergy measured on one benchmark suite can flip sign on an equally reasonable, structurally different distribution, and nothing in a single-suite measurement would reveal that. Accordingly: **a claimed interaction effect does not count as a result unless its sign is consistent across at least three structurally different task distributions.** Below that bar it is a benchmark-suite artifact, not a finding, regardless of how large it measures on any one suite.

The hypotheses in §19 are not mutually independent, and treating them as if they could be tested in any order or in parallel was an error in the source draft. The actual dependency structure:

```
H2 (persistent state) ─┬─→ H3 (adaptive representation) ─→ H8 (representation synergy)
                        └─→ H5 (memory selection)
H1 (predictive state) ──────────────────────────────────→ H7 (model revision)
H4 (adaptive computation) ───────────────────────────────→ H6 (active perception)
```

If H2 fails — persistence provides no measurable benefit — then any experiment testing H3, H5, or H8 on top of a persistent-state implementation is testing a broken foundation, and a positive result for H3 would be uninterpretable, not merely weaker. **H2 must pass before H3, H5, or H8 are run at all**, not be tested alongside them for convenience.

---

## 19. The hypothesis matrix — primary vs. exploratory

**Primary (pre-registered, confirmatory — the program stands or falls on these first):**

| ID | Hypothesis | Null | Gate |
|---|---|---|---|
| H2 | Persistent state reduces compute-per-useful-decision vs. reconstruction | No measurable reduction | None  tested first |
| H1 | Compact predictive state outperforms observation-heavy processing on long-horizon prediction/control | No advantage after controlling for parameter count and data | Requires H2 |
| H7 | Explicit error diagnosis improves adaptation to changed dynamics more than replay/memorization alone | No advantage | Requires H1 |

**Exploratory (informative but not confirmatory until independently pre-registered and re-run):**

H3 (adaptive representation), H4 (adaptive computation), H5 (memory selection), H6 (active perception), H8 (representation synergy). These may motivate the next primary round; a positive exploratory result is not evidence on its own.

This split exists because running all eight as if equally confirmatory, against a shared pool of benchmark variants, produces a multiple-comparisons problem: with enough comparisons, several will show "significant" advantage by chance alone (Ioannidis, 2005, *PLoS Medicine*, on exactly this failure mode in fields with many researcher degrees of freedom). Only the three primary hypotheses are protected against this by being fixed in advance; everything else must be treated as hypothesis-generating, not hypothesis-confirming, until it graduates to its own pre-registered test.

---

## 20. Falsification criteria — no post-hoc redefinition permitted

The prior draft's criteria used "sufficiently demanding benchmarks" as the threshold for failure — this cannot be triggered in practice, because a disappointing result can always be attributed to the benchmark being insufficiently demanding after the fact. Replaced with a firm requirement:

**Before any Stage 0 data is collected, the following must be fixed in writing and not altered afterward:** the exact task, the exact metric, the exact sample size, and the exact numeric threshold that constitutes a pass for H2 (per §19, the first gate). This is the Registered Reports discipline (used in psychology and biomedicine specifically to prevent this failure mode). Absent a pre-registered threshold, no claim in this paper about H2 passing or failing should be considered evidence, in either direction.

Given a pre-registered threshold, the program is weakened if: H2 fails at that threshold; H1 or H7 fail once H2 has passed; or a homogeneous baseline matches the primary-hypothesis system's pre-registered metric at equal or lower resource cost on the pre-registered task.

---

## 21. Relationship to current research — verified precedent, not competitors

| System | Demonstrates | Source |
|---|---|---|
| V-JEPA 2 | Zero-shot physical robot planning from self-supervised video, no task-specific reward | Assran et al., arXiv:2506.09985, 2025 |
| DreamerV3 | Compact world models supporting planning-through-imagination, 150+ tasks, fixed hyperparameters | Hafner et al., *Nature* 640:647–653, 2025 |
| Genie 2 | Action-controllable latent environments | DeepMind technical report, Dec. 2024 — not peer-reviewed |
| Mamba | Selective state-space sequence modeling as attention alternative | Gu & Dao, arXiv:2312.00752, 2023; COLM 2024 |
| HDC/VSA | Distributed, compositional, associative representation | Kleyko et al., *ACM Computing Surveys* 55(6):130, 55(9):175, 2022/2023 |
| Event-based vision | Asynchronous, high-temporal-resolution sensing | Gallego et al., *IEEE TPAMI*, 2020 |

These occupy different points in the hypothesis space defined above. None competes for a single "AGI architecture" title, and this paper makes no claim that any combination of them constitutes one.

---

## References

De Lange, M. et al. (2022). A continual learning survey: defying forgetting in classification tasks. *IEEE TPAMI.*
Gallego, G. et al. (2020). Event-based vision: a survey. *IEEE TPAMI.*
Gu, A. & Dao, T. (2023). Mamba: linear-time sequence modeling with selective state spaces. *arXiv:2312.00752*; COLM 2024.
Hafner, D., Pasukonis, J., Ba, J. & Lillicrap, T. (2025). Mastering diverse control tasks through world models. *Nature*, 640, 647–653.
Hauser, A. & Bühlmann, P. (2012). Two optimal strategies for active learning of causal models from interventions. *International Journal of Approximate Reasoning.*
Hyttinen, A., Eberhardt, F. & Hoyer, P. (2013). Experiment selection for causal discovery. *JMLR.*
Ioannidis, J. (2005). Why most published research findings are false. *PLoS Medicine.*
Kleyko, D., Rachkovskij, D., Osipov, E. & Rahimi, A. (2022, 2023). A survey on hyperdimensional computing aka vector symbolic architectures, Parts I & II. *ACM Computing Surveys*, 55(6):130, 55(9):175.
Parker-Holder, J. et al. (2024). Genie 2: a large-scale foundation world model. Google DeepMind technical report.
Assran, M. et al. (2025). V-JEPA 2. *arXiv:2506.09985.*
Snell, C., Lee, J., Xin, K. & Kumar, A. (2024). Scaling LLM test-time compute optimally can be more effective than scaling parameters. *arXiv:2408.03314.*
Tishby, N. & Zaslavsky, N. (2015). Deep learning and the information bottleneck principle. *IEEE ITW.*
Vaswani, A. et al. (2017). Attention is all you need. *NeurIPS.*
Wolpert, D. & Macready, W. (1997). No free lunch theorems for optimization. *IEEE Transactions on Evolutionary Computation.*
