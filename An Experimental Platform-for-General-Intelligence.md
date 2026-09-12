# Paper II An Experimental Platform for General Intelligence: Open-World Agents, Computational Substrates, and Hardware Co-Design

**Status:** Engineering research program. **Depends on:** Paper I.

---

## Abstract

A system intended to test theories of general intelligence cannot be reduced to benchmark scores from static datasets. This paper specifies a platform for continuous interaction, state maintenance, and prediction under uncertainty, with internal state exposed for measurement rather than treated as a black box. It answers exactly one question at this stage — does persistent state (Paper I, H2) reduce redundant computation without degrading control — before any larger claim is entertained, and everything past that point is explicitly left undesigned until that question has a pre-registered answer.

---

## 1. Engineering principle

Not "build an AGI" — too vague to falsify. An **experimental platform** with independently switchable mechanisms, each requiring a defined interface, measurable resource cost, measurable behavioral effect, reproducible implementation, and controlled ablation:

```text
Agent
├── perception ├── representation ├── world model ├── memory
├── uncertainty ├── planner ├── adaptive-compute controller
├── tool interface ├── self-model └── learning/revision
```

Only the components needed for Stage 0 (perception, a minimal persistent-state module, a reconstruction-baseline module) are built now. The rest exist in this diagram as a target, not a current build requirement.

---

## 2. Experimental philosophy

**Principle 1 — same task, different computation.** **Principle 2 — same capability, different substrate; never claimed until §21's profiling stage is reached.** **Principle 3 — measure energy, not only accuracy.** **Principle 4 — test distribution shift explicitly.** **Principle 5 — falsification must be possible, which requires pre-registration (§9), not just a stated intention to be falsifiable.**

---

## 3. Stage 0 environment — the only environment specified in this document

A small, fully-specified environment: a 2D or simple 3D grid world with a handful of manipulable objects, one hidden environmental parameter that changes object dynamics at an unannounced point mid-episode (e.g., friction), and complete simulator-side ground truth. This is deliberately not Isaac Sim, not a difficulty ladder, not a multi-domain benchmark — those are Stage 1+ concerns, undesigned here. The only requirement on this environment: it must be able to falsify or confirm H2 (persistence reduces compute-per-decision without degrading control) and, contingent on that, H1 and H7 per Paper I §18–19's dependency order.

---

## 4. Digital twin

Ground truth $S_t^{GT}$ vs. agent estimate $\hat{Z}_t$, compared directly on pose, velocity, and the hidden dynamics parameter specifically — this is the one comparison Stage 0 exists to make measurable.

---

## 5. Stage 0 agent

```text
Sensors → Perception → { B1: dense neural state, or B2: incrementally-updated persistent state }
        → Planner/Controller → Actions → Environment
```

Only these two conditions are specified. Everything past B2 (structured world state, multirepresentation, adaptive compute) is Stage 1+ and is not designed until B1 vs. B2 produces a pre-registered result.

---

## 6. What is deliberately not yet specified

The full difficulty ladder (partial observability, novel objects, novel dynamics, nonstationarity, long horizons, multiscale dynamics, novel compositions, active exploration, open-world robotics), the full baseline ladder past B2, and the full hardware ladder past a CPU/GPU baseline are **not designed in this document.** Designing memory budgets, scheduler complexity, and accelerator roadmaps for mechanisms that have not yet been shown to work at Stage 0 was the original draft's central overreach. They will be specified, one stage at a time, only once the preceding stage has a pre-registered result — a positive or negative one, both are informative.

---

## 7. Distribution-shift protocol (Stage 0 scope only)

At Stage 0, exactly one novelty axis is tested: the hidden dynamics change (mechanistic novelty). Geometric, object, sensor, behavioral, and compositional novelty axes are real and eventually necessary but are explicitly out of scope until Stage 0 passes.

---

## 8. Metrics (Stage 0)

Task success $S$; prediction error $E_p = d(Z_{t+\tau}, \hat{Z}_{t+\tau})$; compute-per-decision; energy-per-decision; and detection latency for the hidden dynamics change. **No composite weighted metric is used at this stage** — the original draft's $\eta_I$ composite, with weights set by the experimenter, is deferred until there is enough data across multiple primitive metrics to justify combining them, and even then the weights must be pre-registered, not fitted to produce a preferred ranking.

---

## 9. Pre-registration requirement

Per Paper I §20: before Stage 0 data collection begins, the exact task, exact metric (from §8), exact sample size, and exact numeric pass/fail threshold for H2 must be fixed in writing and not altered afterward. Any result reported without this having been done in advance should be read as exploratory, not confirmatory, regardless of how the result is framed.

---

## 10. Experiments : primary vs. exploratory

**Primary (Stage 0, pre-registered, run first):**

**Experiment 1 — Persistent vs. reconstructed state.** $O_{1:t} \to \text{reconstruct} \to A_t$ vs. $O_t \to \text{incremental update} \to Z_t \to A_t$. Prediction: $E_{\text{persistent}} < E_{\text{reconstruct}}$ specifically when only a small fraction of the world changes per step. This is the single experiment the rest of the program is gated on.

**Experiment 8 — Model revision under a changed physical parameter.** $\mu_{\text{friction}}: 0.5 \to 1.2$ mid-episode. Measures detection latency, adaptation time, transient failure rate, final performance. This is the sharpest available test of whether the system revises models or merely fits behavior, and it is kept as primary because it is cheap to run alongside Experiment 1 in the same Stage 0 environment and tests H7, the third primary hypothesis.

**Exploratory (informative, not confirmatory, deferred until Experiment 1 and 8 produce pre-registered results):**

Representation selection, event-triggered computation, adaptive compute via value-of-information, active sensing, continual learning across sequential tasks. Each of these presupposes persistence and/or model revision already work — running them before Experiment 1/8 conclude would test H3/H5/H8-type claims on top of an unvalidated foundation, exactly the dependency violation flagged in Paper I §18.

---

## 11. From software experiment to hardware experiment : deferred by design

Hardware profiling and any hardware ladder are **not part of this document's current scope.** Given a primitive set $\mathcal{P}$, one would eventually measure each primitive's share of runtime ($\rho_i$) and energy ($\epsilon_i$) and specialize only high-$\rho_i$, high-$\epsilon_i$ operations — the same discipline argued for on independent grounds (Roofline arithmetic-intensity analysis; Amdahl's Law) in the companion engineering work. But this requires a working Stage 1+ system with a measured profile, which does not yet exist. Naming candidate hardware now, before that profile exists, would repeat the exact error this revision is correcting.

---

## 12. The decisive comparison : corrected

The original draft's criterion was a binary equalization: "equal task performance at lower energy/latency" or "equal resources with higher OOD generalization," with an unspecified numeric bar for what counts, and no defined exchange rate between compute units across substrates. Both defects are corrected here:

**No binary equalization is used.** Instead, for any two systems being compared, report the full empirical frontier — performance, energy-per-decision, and latency-per-decision, each measured independently, with no forced conversion between them. A system is preferred only if it **Pareto-dominates** the alternative (strictly better on at least one axis, no worse on any other), measured against the pre-registered thresholds from §9. If neither system dominates — one is more accurate, the other more efficient — that is reported as an honest trade-off, not resolved by inventing a weighted score after the fact to produce a winner.

---

## 13. Sim-to-real, with the limitation named rather than hidden

Domain randomization (lighting, friction, sensor noise, actuator response, object mass) varies parameters *within* the simulator's physics model. **It cannot generate novelty outside that model's ontology** — a simulator without soft-body or fluid dynamics cannot produce a domain-randomized test of either, no matter how many parameters are varied. This is a real, currently uncorrected limitation of simulation-first validation, not a solved problem, and any physical-robot stage (deferred, per §6) must be specifically designed to hunt for failures the simulator structurally could not have produced — replicating simulated tests on hardware does not address this gap.

---

## 14. The complete engineering loop, as far as this document specifies it

$$\text{Theory (Paper I, H2 gate)} \to \text{Stage 0 (this paper, §3–10)} \to \text{pre-registered result} \to \text{[everything past this point undesigned until the result exists]}$$

---

## References

(Shared references with Paper I are not repeated.)
