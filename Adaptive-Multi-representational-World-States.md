# Paper III Adaptive Multirepresentational World States: Memory, Prediction, and Value-of-Information Computation

**Status:** Architectural hypothesis and experimental design. **Depends on:** Papers I and II. **Contingent on:** Paper I §18's H2 gate passing at Stage 0 (Paper II §10) before any experiment in this paper is run.

**Revision log — ruthless pass, what was cut and why:**
- **Cut** "the deepest open problem" closing section. It restated the conjunction of hypotheses already listed in §11 in more dramatic language and added no new testable content — the exact pattern already cut from Paper I.
- **Added** an explicit dependency statement: every experiment in this paper presupposes persistent state already works. If Paper I's H2 fails at Stage 0, the entire multirepresentational apparatus below is testing on top of a broken foundation, and any positive result would be uninterpretable, not just weaker. This paper is therefore reclassified in full as **Stage 1+, exploratory, undesigned-in-detail until Stage 0 passes** — consistent with Paper II's collapse of its own ladders.
- **Tightened** the hypothesis list (§11) to remove vague thresholds ("consistently dominates," "sufficiently diverse shift") and require the same pre-registration discipline as Papers I and II before any of these can be treated as confirmatory.
- **Kept** §6 (the substrate-mismatch argument) largely as previously bounded, since it already separated verified evidence from the claim it supports and did not overreach on inspection — this is the one section of the three papers that held up under the stricter pass without needing a cut, only tightening of scope language.

---

## Abstract

An embodied agent cannot afford to reconstruct all of reality at maximum fidelity every timestep. This paper's hypothesis: a persistent world state composed of multiple task-selected representations, with memory and computation allocated by value of information, outperforms a single homogeneous representation under resource constraints. **This entire hypothesis is downstream of, and untestable independent of, Paper I's persistence result (H2).** Section 6 additionally argues, with verified hardware evidence, that some of the operations this requires are measurably mismatched to dense tensor accelerators — a narrower and better-supported claim than "embodied intelligence needs new hardware" stated without qualification.

---

## 1. The physical computation problem

Most of a physical scene is unchanged between consecutive timesteps. Treating every timestep as requiring full reconstruction works against real-time, energy-constrained operation — this is the same redundancy Paper I's H2 is designed to test, and this paper's entire apparatus is built on the assumption that H2 is true.

---

## 2. Persistence as a computational principle (inherited, not re-tested here)

$Z_{t+1} = F(Z_t, \Delta O_t, A_t)$ when $\|\Delta Z_t\| \ll \|Z_t\|$. This paper does not re-test whether persistence works — that is Paper II §10, Experiment 1. It assumes a positive result and asks what should be persisted.

---

## 3. A persistent, structured world state (hypothesis, not yet built)

$$Z_t = (E_t, G_t, R_t, D_t, U_t, S_t, M_t)$$ — entities, geometry, relations, dynamics, uncertainty, semantics, memory references. This structure is not implemented or tested until Stage 1, per the gating above.

---

## 4. Multiple representations, none privileged a priori

$$\mathcal{R} = \{R_N, R_G, R_O, R_R, R_D, R_H, R_S\}$$ maintained selectively, $Z_t^{(k)} = F_k(Z_t, O_t, Q_t)$, rather than all at maximum resolution.

---

## 5. Representation selection as an explicit policy

$$r_t = \pi_R(Z_t, Q_t, U_t, \Delta Z_t, R_t)$$

Geometry for collision, object+relation+semantic for "who is carrying this," dynamics for "will it fall," associative+episodic for recall, neural for ambiguous input.

---

## 6. The substrate question : held to the stricter standard

**The claim, precisely bounded.** Dense tensor accelerators achieve throughput via high arithmetic intensity. Associative lookup ($R_H$), sparse graph update ($R_R$), and event-driven sensing have the opposite profile — each byte typically used once, in a branchy or sparse pattern. This is the same arithmetic-intensity argument (Roofline model) used elsewhere in this program for causal-graph workloads, applied here to associative/relational operations.

**Verified, not analogical, evidence:**

- **Memory-centric:** Karunaratne, Le Gallo, Cherubini, Benini, Rahimi & Sebastian (2020, *Nature Electronics* 3:327–337) — complete in-memory HDC system, 760,000 phase-change memory devices, language/news classification and EMG gesture recognition at software-comparable accuracy, write-once memory.
- **Event-driven:** Intel Hala Point (1,152 Loihi 2 processors, 1.15B neurons, 140,544 cores, Sandia National Laboratories, April 2024) — up to 15 TOPS/W on conventional DNN inference **without batching**, a direct answer to a real-time constraint batching-dependent architectures cannot satisfy by construction (Intel Newsroom, 2024; vendor-reported, not independently benchmarked).

**What this does not establish, stated as firmly as the positive claim:** neither result shows an embodied agent's *entire* stack should leave dense tensor hardware. $R_N$ and any learned continuous function approximator remains well matched to GPU arithmetic intensity. This section motivates why the question is worth Stage-2-or-later profiling (Paper II §11, itself deferred); it does not pre-empt that profiling's answer.

---

## 7. Geometry is necessary but not sufficient

$x, y, z$ alone does not answer "what happens next" — needs velocity, transition probability, causal relation, uncertainty layered on. Geometry $\neq$ world model.

---

## 8–9. Multiscale representation (hypothesis, Stage 1+)

$Z_t = \{Z_t^{\text{fast}}, Z_t^{\text{medium}}, Z_t^{\text{slow}}\}$ at different native frequencies. Untested until the base persistence result exists.

---

## 10. Memory as an active hierarchy (hypothesis, Stage 1+)

$$M_t = \{M_w, M_e, M_s, M_p, M_c\}, \qquad \text{Score}(x) = \alpha N(x) + \beta P(x) + \gamma U(x) + \delta C(x) + \epsilon V(x) - \lambda\, \text{Cost}(x)$$

Prediction failure ($\epsilon_t \gg \mathbb{E}[\epsilon]$), not novelty alone, should drive storage priority — a specific, testable claim once Stage 1 begins, not before.

---

## 11. Hypotheses : tightened, pre-registration required, all contingent on Stage 0

| ID | Hypothesis | Corrected null / bar |
|---|---|---|
| H1 | Persistence + multirepresentation reduces compute-per-decision vs. persistence alone | No reduction at the pre-registered threshold (TBD before data, per Paper I §20 discipline) |
| H2 | Dynamic representation selection ($\pi_R$) outperforms any single fixed representation | Fixed representation matches or beats selection at equal budget |
| H3 | Selective, prediction-failure-triggered memory outperforms indiscriminate retention | Indiscriminate retention matches or beats it at equal memory budget |
| H4 | Value-of-information-gated compute improves the performance–energy frontier (Pareto sense, per Paper II §12) vs. fixed compute | No Pareto improvement |

All four are **exploratory relative to this whole research program** until Stage 0 (Paper II) passes : none is confirmatory before then, and none should be reported as if it were. The original eight-hypothesis list is cut to four; the remainder (event-triggered computation, active sensing, task-conditioned representation, compression-vs-control tradeoffs) are real questions but are folded into "future work once Stage 1 exists" rather than listed as if currently testable.

---

## 12. Relationship to world models : the actual open question

V-JEPA 2 and DreamerV3 each show one compact learned representation suffices for their tested domains. This paper's question — one model vs. a selectively-activated collection — remains open and is not resolved by either system's existence, since neither was built to test the comparison.

---

## 13. Relationship to Transformers
 A Transformer remains a valid candidate implementation of $R_N$. Different operations may need different substrates, which is about *where* computation runs, not whether any current architecture is sufficient.

---

## References

(Shared references with Papers I and II are not repeated.)

Karunaratne, G., Le Gallo, M., Cherubini, G., Benini, L., Rahimi, A. & Sebastian, A. (2020). In-memory hyperdimensional computing. *Nature Electronics*, 3, 327–337.
Intel Corporation (2024). Intel builds world's largest neuromorphic system to enable more sustainable AI. Intel Newsroom, April 17, 2024.
