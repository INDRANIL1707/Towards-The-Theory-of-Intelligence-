# Memory Beyond Storage

## A Computational Hypothesis of Memory as State-Dependent Dynamics, and a Program to Test It

**Status:** Research hypothesis / exploratory framework — revised for terminological precision, honest positioning against prior work, and experimental grounding.
**Purpose:** Formalization and falsification program for theoretical and engineering investigation.
**Keywords:** memory, persistent state, POMDP belief states, recurrent/state-space models, associative memory, test-time learning, in-memory computing, AGI, metamemory.

---

### Note on this revision

The original draft made two kinds of claims without distinguishing them: (1) a *weak* claim — persistent history-dependent state changes future computation — which is already the formal definition of a state-space system and is not new; and (2) a *strong* claim — that memory should be understood as structure which reconfigures the transition operator itself, not merely as an extra input consumed by a fixed operator — which is a live, unresolved, and actively-researched question. The original text also boxed the same restated claim five or six times, used `m_t` and `M_t` inconsistently, and did not cite the substantial existing literature that already formalizes most of the "weak" claim. This revision:

1. Fixes notation.
2. Adds a related-work section that names what is already established, so the residual novel claim is visible instead of implied by omission.
3. Sharpens the "is this just state?" objection with a proposed additional criterion, rather than leaving it as rhetorical.
4. Collapses repeated boxed restatements into single, load-bearing statements.
5. Replaces the abstract "Model A / B / C" sketch with named, currently-buildable architectures, existing benchmarks, and operational metrics — including a weekend-scale experiment.
6. Adds an explicit connection to the CAF architecture and hardware-substrate research track, since that program is where this hypothesis would actually be tested and used.

---

## Abstract

Memory is conventionally treated as the storage and later retrieval of information acquired in the past. This paper examines whether that description is adequate for intelligent systems, and argues that it is not, for a specific and testable reason.

The weak version of the alternative view — that persistent history-dependent state affects future computation — is simply a restatement of what a state-space system is (§3–4), and is already formalized in control theory (belief states), recurrent network theory, and information theory. The strong version — that memory should be understood as structure which *reconfigures the transition operator* $F$ itself, rather than as an additional input consumed by a fixed $F$ — is not settled, and is the actual subject of current machine learning research on test-time-updated ("fast-weight") memory. This paper's contribution is to state that distinction precisely, show that most existing frameworks (RNNs, transformers with retrieval, state-space models) instantiate only the weak version, and propose a concrete, falsifiable, currently-runnable experimental program that can determine whether the strong version produces a measurable, not merely definitional, difference in a system's computational behavior.

$$
\boxed{\textbf{Central hypothesis: memory is persistent, history-dependent state whose presence or absence changes the transition operator } F \textbf{ of an intelligent system, not merely the values } F \textbf{ operates on.}}
$$

The central empirical question this reframes:

> Does a system in which memory directly reconfigures $F$ (rather than augmenting $F$'s input) produce measurably different computational behavior — reachable-state diversity, graceful degradation under partial cues, interference-vs-plasticity trade-offs, or efficiency at matched compute — than one in which memory is a literal store consulted by a fixed $F$?

Section 14 gives a concrete way to answer this with existing tools and benchmarks.

---

## 1. Notation

To avoid the inconsistency in the original draft, notation is fixed as follows for the remainder of this document:

- $x_t$ — fast/working computational state (analogous to an RNN's hidden state at a single layer, or a controller's internal state).
- $M_t$ — the full persistent memory state at time $t$ (a set, vector, matrix, or parameter tensor, depending on architecture).
- $m \in M_t$ — an individual memory trace or item within $M_t$ (used only when addressing individual items, e.g. in discussing forgetting of a specific trace).
- $H_t = (o_1, a_1, \ldots, o_t)$ — the full interaction history up to time $t$.
- $c_t$ — a retrieval cue at time $t$ (may be external or internally generated).
- $F$ — the system's transition operator, $x_{t+1} = F(x_t, u_t)$ in the state-only case, or $F_{M_t}(x_t, u_t)$ when memory is claimed to reconfigure $F$ itself (the strong claim; see §5).
- $\phi$ — a compression/encoding map from history or observation to state, $z_t = \phi(H_t)$.

---

## 2. Introduction

The common computational metaphor for memory is:

$$
\text{experience} \rightarrow \text{storage} \rightarrow \text{retrieval} \rightarrow \text{use}.
$$

This metaphor separates memory from computation: computation happens in one place, memory sits in another, and a retrieval operation moves information between them. It is a reasonable description of many engineered storage systems (a filesystem, a hash table, a vector database), but it is a poor description of at least one thing everyone has experienced directly.

A person cannot recall the name of someone they know. They experience the distinct state of "knowing that they know" without being able to produce the item. A few moments later, an unrelated remark, a photograph, or an internally generated association makes the name available. Nothing was re-stored in the interim.

The phenomenon of interest is not that the information existed somewhere. It is:

$$
\text{same agent} + \text{different cue/context} \rightarrow \text{different accessible knowledge}.
$$

This licenses a distinction between *stored*, *accessible*, and *actionable* information, which need not coincide. The question this paper asks is not "where is the memory," but:

> What computational role does persistent state play in an intelligent system, and is that role better described as storage-plus-lookup, or as something that changes the system's transition dynamics?

---

## 3. Related Work: Isolating What Is Actually New

Before proposing a "computational fabric" hypothesis, it is necessary to state plainly how much of it already exists in formalized, well-tested form elsewhere. Presenting these ideas without this section would overstate the paper's novelty.

**Belief states and sufficient statistics (control theory, POMDPs).** The claim in §6–8 that a compressed representation $z_t = \phi(H_t)$ should preserve only task-relevant structure from history is the classical notion of a *sufficient statistic* or *belief state* for a partially observable process, formalized since Kalman filtering (1960) and standard in POMDP theory. This part of the framework is not new; it is a restatement of 60-year-old control theory in different vocabulary.

**Recurrent and state-space models.** Any recurrent network — Elman networks, LSTMs (Hochreiter & Schmidhuber, 1997), and modern structured state-space models such as S4 (Gu, Goel & Ré, 2021–22) and Mamba (Gu & Dao, 2023) — already satisfies "persistent state changes future computation" by definition: the hidden state is literally part of $F$'s input. The paper's §4.2 ("memory as persistent state") and the weak reading of §4.3 describe nothing beyond the standard definition of a state-space system. This is the paper's own §22 objection, and it is correct: without a further criterion, "memory" as originally defined does not pick out anything distinct from "state."

**Fast-weight and test-time-updated memory.** The *strong* claim — that memory should reconfigure the transition operator itself, $F_{M_t}$, rather than augment its input — has concrete prior instances. Fast Weight Programmers (Schmidhuber, 1992) and "Using Fast Weights to Attend to the Recent Past" (Ba, Hinton, Mnih, Leibo & Ionescu, 2016) use a slow network to produce weight changes for a fast associative network, and those fast weights participate directly in subsequent computation rather than being looked up. More recently, Titans (Behrouz, Zhong & Mirrokni, Google Research, arXiv:2501.00663, submitted Dec. 2024) introduce a neural long-term memory module that updates its own parameters at test time — driven by a gradient/"surprise" signal — and those parameters directly enter the forward computation on subsequent input. This is the closest existing working instance of the strong claim in §5, and it is recent, active research, not settled science.

**Associative memory and attractor dynamics.** §7's description of retrieval as "cue-driven navigation toward a useful reconstructed state" is precisely the classical Hopfield network picture (Hopfield, 1982), formalized with an explicit energy function and exponential storage capacity by Modern Hopfield Networks (Ramsauer et al., 2020). Retrieval-as-convergence-to-an-attractor is an established, quantitatively developed framework, not a new proposal.

**External differentiable memory.** Neural Turing Machines (Graves, Wayne & Danihelka, 2014) and the Differentiable Neural Computer (Graves et al., *Nature*, 2016) built explicit, differentiable read/write memory matrices coupled to a controller a decade before this paper. They mostly use content-addressed *lookup*, which places them closer to the weak claim than the strong one, but they are a direct engineering precedent for "memory participates in computation" as an architectural principle.

**Complementary Learning Systems (CLS) theory.** McClelland, McNaughton & O'Reilly (1995) give the neuroscience account of why biological memory needs at least two timescales — a fast, high-plasticity system (hippocampus) and a slow, consolidating system (cortex) — which grounds §10's multi-timescale claim and supplies a principled answer to "why not just one $M$."

**Reservoir computing.** Jaeger's (2001) *memory capacity* measure for echo-state networks — $MC = \sum_{k=1}^{\infty} \mathrm{corr}^2\big(y_k(t), u(t-k)\big)$, where $y_k$ is a linear readout trained to reconstruct the input delayed by $k$ steps — is an existing, quantitative answer to "how much of history survives compression into $M_t$," and is directly reusable as an operational metric (§14.1).

**Information Bottleneck.** Tishby, Pereira & Bialek (1999) formalize "compress history while retaining only task-relevant structure" (§8's claim) as the explicit objective $\min I(Z;X) \text{ s.t. } I(Z;Y) \ge \text{const}$.

**What is not yet settled.** None of the above resolves whether a system in which memory *literally reconfigures* the transition operator produces a *categorically* different class of computation — different reachable-state geometry, different generalization behavior, different graceful-degradation properties, different efficiency at matched compute — from a system in which memory is a fixed operator's input (however well-compressed or content-addressed that input is). That is the residual, genuinely open part of the hypothesis, and it is what Section 14's experimental program is designed to probe. Everything in §4–§11 below should be read with this positioning in mind: much of it names existing ideas in a common vocabulary; the load-bearing novel claim is narrow, and clearly marked in §5.

---

## 4. From Storage to State

The naïve model treats memory as a repository, $M = \{m_1, \ldots, m_n\}$, with retrieval $R(M, q) \rightarrow m_i$. This assumes remembering consists of locating an existing object. It cannot easily explain why $\text{stored}(m) = 1$ does not imply $\text{accessible}(m) = 1$ — the tip-of-the-tongue case from §2.

A model consistent with cue-dependence is:

$$
\hat m_t = R(M_t, c_t, x_t, g_t)
$$

where $g_t$ is the current goal. This reframes the research question from "where is the memory stored" to "what conditions make past-derived structure accessible to present computation" — which is the belief-state question already addressed in POMDP theory (§3), stated in different language.

---

## 5. Memory as Modified Dynamics: The Real Fork

This is the section that carries the paper's actual claim, separated explicitly into weak and strong versions.

**Weak version (established).** Persistent state $M_t$ is consumed as an input to a fixed transition operator:

$$
x_{t+1} = F(x_t, M_t, u_t), \qquad M_{t+1} = G(M_t, x_t, u_t).
$$

Every RNN, SSM, and retrieval-augmented transformer satisfies this. It is true, and it is not new (§3).

**Strong version (open, the actual hypothesis).** Persistent state parametrizes the operator itself:

$$
x_{t+1} = F_{M_t}(x_t, u_t), \qquad M_{t+1} = G(M_t, x_t, u_t).
$$

Here, two agents with identical $(x_t, u_t)$ but $M_t^{(1)} \neq M_t^{(2)}$ do not merely receive different inputs to the same function — they compute with *different functions*: $F_{M_t^{(1)}} \neq F_{M_t^{(2)}}$ as operators, not just as outputs. Learning changes $M_t \rightarrow M_{t+1}$ and therefore changes which function the system computes with, not only what value that function returns.

$$
\boxed{\text{Weak claim: memory is state consumed by computation. Strong claim: memory is structure that reconfigures computation. Only the strong claim is a live research question.}}
$$

Fast-weight programmers and Titans (§3) are the closest existing instances of the strong version. Whether the strong version yields behavior that is *categorically* distinguishable from a well-designed instance of the weak version (e.g., a large SSM with a well-compressed state) — rather than just a difference of degree — is unresolved, and is the target of §14.

---

## 6. Partial Observability and Sufficient Statistics

If an agent's observations $o_t$ underdetermine $P(o_{t+1} \mid o_t)$, it needs history-dependent information: $P(o_{t+1} \mid o_{\le t}, a_{\le t})$. The computational problem is to compress $H_t$ into some $z_t = \phi(H_t)$ that is *sufficient* — preserving what is needed for prediction, inference, control, and action, not the raw history itself. This is the belief-state formalization noted in §3; nothing here is specific to biological memory. The reframing worth keeping is negative: an intelligent system rarely needs to preserve raw history in full, and the interesting design question is which structure must survive compression, not how much can be stored.

---

## 7. Retrieval as Reconstruction and Navigation

Under the storage metaphor, $\text{query} \rightarrow \text{stored record}$. Under the reconstruction view:

$$
\hat z_t = R(M_t, c_t, x_t, g_t)
$$

is a task-optimized reconstruction, not necessarily an exact historical copy — consistent with the well-established finding in cognitive psychology that recall is reconstructive rather than a literal replay of an immutable record.

A cue can be modeled as a perturbation $x_t \rightarrow x_t'$ that the system's dynamics then relax toward a stable region: $x_t' \rightarrow x_{t+1} \rightarrow \cdots \rightarrow A_i$. As noted in §3, this is exactly the Hopfield/Modern-Hopfield attractor picture, with an existing energy-function formalization and provable capacity bounds — not a new proposal, but a useful and well-supported one to adopt.

The one prediction this view makes that a pure lookup table does not: retrieval accuracy should degrade *gracefully* as the cue is corrupted (partial match), rather than falling off a cliff at the first non-matching bit. This graceful-degradation signature is the operational test used in §14.

---

## 8. Forgetting as Computational Control

Forgetting need not mean physical deletion. Alternatives, none mutually exclusive:

- **Accessibility loss** — stored but temporarily unreachable ($\text{stored}=1$, $\text{accessible}=0$).
- **Interference** — competing structures reduce retrieval selectivity.
- **Relevance-based degradation** — low-utility information is weakened preferentially.
- **Retrieval-pathway degradation** — the trace persists, but the cues/associations needed to reach it are lost.
- **State-space restructuring** — the reachable regions of the state space shift, changing what is easy to reach at all.

If retention and retrieval have cost $C(M)$ and utility $U(M)$, a plausible objective is $\max_M [U(M) - \lambda C(M)]$, under which forgetting is an adaptive control mechanism rather than a failure mode. This reframing is useful, but it is worth being honest that it is a hypothesis about *why* biological forgetting might be adaptive, not a demonstrated mechanism — the "which information gets forgotten and why" question in real biological or artificial systems remains open and system-specific.

---

## 9. Metamemory

A distinct phenomenon: an agent can represent that it cannot currently retrieve something ("I know this person, but not their name"). This requires the agent to hold a state like $q(m) = $ "a memory of $m$ is expected but currently inaccessible" — a model of the agent's own retrieval process, not just of the world. This produces the recursive structure: memory → model of memory → detection of retrieval failure → search for a better cue → reconstruction. Whether artificial systems need an explicit analog of this (rather than merely a well-calibrated confidence signal on outputs) is an open design question relevant to uncertainty-aware agents, and is listed as RQ8 in §13's antecedent (Research Questions carried over from the original draft, condensed): what confidence/accessibility signal, if any, must be represented separately from the retrieved content itself.

---

## 10. Multi-Timescale Architecture

Biological and engineered systems both suggest memory is not one mechanism but several, organized by persistence timescale:

$$
\tau_1 \ll \tau_2 \ll \tau_3 \ll \cdots, \qquad M = \{M_1, \ldots, M_n\}.
$$

Complementary Learning Systems theory (§3) is the established grounding for at least a two-timescale split (fast/plastic vs. slow/consolidating). The open architectural question is not how many layers to have, but *which timescale should be permitted to reconfigure $F$ itself* (§5's strong claim) versus which should only ever supply input to a fixed $F$ — this is the practical design fork explored in §14.

---

## 11. Is This Just "State"? Sharpening the Criterion

The most serious objection to this entire framework, raised honestly in the original draft (§22) but not resolved there: any dynamical system already has state, $x_{t+1} = F(x_t, u_t)$. If "memory" is just relabeled state, the theory is a semantic exercise, not a substantive claim.

The original draft's proposed criterion —

$$
\text{Memory} = \text{persistent state whose present value depends on past interaction and whose persistence changes future computation}
$$

— does not actually resolve the objection: every RNN hidden state satisfies it trivially. A functional test based on this criterion (two histories, same immediate observation, different persistent state, different future behavior) is likewise satisfied by any stateful system, including a plain LSTM. The criterion needs at least one more condition to have teeth.

**Proposed additional criterion (this revision's contribution): timescale separation.** Require that memory be distinguishable from working state by evolving on a slower characteristic timescale, $\tau_M \gg \tau_x$, consistent with CLS theory (§3, §10) — and, for the strong claim (§5), additionally require that $M_t$ enter as a *parameter* of $F$ rather than as a *co-argument* alongside $x_t$. Under this sharper criterion:

- A plain RNN's hidden state is *not* memory in the strict sense (no timescale separation from $x_t$ — they are the same variable).
- A slowly-updated retrieval index or KV-cache *is* memory in the weak sense (timescale-separated, but enters as input, so $F$ itself is fixed) — this covers most current "long-context" and RAG-style systems.
- A fast-weight or Titans-style module that both updates on a slower schedule than $x_t$ *and* directly parametrizes $F$ is memory in the strong sense.

This gives a testable typology instead of a single ambiguous label, and is directly operationalized in §14.2's three architectures.

---

## 12. The Central Hypothesis

$$
\boxed{\textbf{Memory is persistent, timescale-separated state whose structure can (in the strong form) reconfigure an intelligent system's transition operator, rather than merely serving as an additional input to a fixed operator. The purpose of memory is not to preserve the past, but to convert past interaction into a change in what the system computes with.}}
$$

This is stated once, deliberately, rather than boxed repeatedly across sections as in the original draft.

---

## 13. Falsifiable Predictions

1. **Divergent dynamics.** Two systems with identical $(x_t, u_t)$ but $M_t^{(1)} \neq M_t^{(2)}$ under the strong-form architecture should exhibit next-step behavior that differs as *functions*, not just as outputs — measurable via divergence in predicted distributions across a matched-input test set, not merely via different single predictions.
2. **State-space change beats capacity increase.** Under a matched parameter/compute budget, an architecture in which memory reconfigures $F$ should sometimes outperform an architecture that merely enlarges the input given to a fixed $F$ (e.g., a larger context window or bigger retrieval index).
3. **Graceful degradation.** Cue-dependent reconstruction (§7) should degrade gracefully under cue corruption applied only at test time, whereas literal storage-and-lookup should show a sharp accuracy cliff at the first non-matching element.
4. **Forgetting–plasticity trade-off.** Systems whose memory reconfigures $F$ at test time should show a controllable trade-off between plasticity (fast adaptation to new information) and stability (resistance to interference), visible as a measurable backward-transfer curve in a continual-learning protocol — not simply "more forgetting is bad."
5. **Learning changes trajectory structure.** Learning should produce a measurable change in the geometry of the system's state trajectories (effective dimensionality, attractor structure), not only a change in downstream accuracy.
6. **Substrate-locality advantage (new, added in this revision).** The efficiency advantage of strong-form memory (relative to weak-form retrieval at matched task performance) should be larger on hardware where memory and computation are physically co-located than on von Neumann hardware, since the latter already pays a data-movement cost regardless of architecture. This connects the computational hypothesis to the hardware-substrate question in §14.6.

---

## 14. Engineering and Experimental Program

This section replaces the original draft's abstract "Model A / B / C" sketch with named architectures, existing benchmarks, and metrics that can be run without inventing new infrastructure.

### 14.1 Operationalizing the quantities

| Quantity from the theory | Operational metric | Source |
|---|---|---|
| "How much history survives compression into $M_t$" (§6, §8) | Memory capacity $MC = \sum_k \mathrm{corr}^2(y_k(t), u(t-k))$ for a linear readout trained to reconstruct delayed input | Jaeger (2001), reservoir computing |
| "Reachable-state diversity" (original §17) | Participation ratio $PR = (\sum_i \lambda_i)^2 / \sum_i \lambda_i^2$ of the covariance of $\{M_t\}$ across a task distribution | Standard dimensionality metric from dynamical-systems / neuroscience analysis |
| "Retrieval as navigation to an attractor" (§7) | Local Jacobian $\partial M_{t+1}/\partial M_t$ along a trajectory; leading eigenvalues indicate contraction (stable/forgetting direction) vs. expansion | Standard dynamical-systems tool |
| "Reconstruction vs. lookup" (§7, Prediction 3) | Retrieval accuracy as a function of test-time-only cue corruption level | Directly measurable in any associative-recall task |
| "Interference / adaptive forgetting" (§8, Prediction 4) | Backward-transfer accuracy: performance on task A after training on task B, no rehearsal | Standard continual-learning protocol |
| Overall utility-per-cost, $\mathcal{V}(M)$ | $U(M)$ = downstream task performance; $C_{\text{memory}}$ = parameter/bit count of the memory module; $C_{\text{retrieval}}$ = FLOPs per query; $C_{\text{computation}}$ = FLOPs to update $M_t$ | Direct instrumentation of any candidate architecture |

### 14.2 Three concrete, currently-buildable architectures

- **Architecture A — Literal store.** A frozen key–value or vector index (e.g., FAISS/ScaNN nearest-neighbor lookup, or an unbounded transformer KV-cache). Retrieval is exact- or nearest-match lookup; $M$ never reconfigures $F$, it only supplies additional context. This is the "weak claim, input-only" case from §5.
- **Architecture B — Compressed recurrent state.** An SSM-class model (GRU, LSTM, S4, or Mamba). History is compressed into a fixed-size state that *is* $x_t$; there is no separate $M_t$ with its own timescale. By the §11 criterion, this does not qualify as memory in the strict sense — it is included as the standard baseline everyone currently compares against.
- **Architecture C — Test-time-parametrized memory.** A fast-weight layer (Schmidhuber, 1992; Ba et al., 2016) or a Titans-style neural long-term memory module, updated on a slower schedule than $x_t$ by its own internal learning rule (e.g., gradient descent on an associative reconstruction loss), whose parameters directly enter $F$ on subsequent steps: $F_{M_t}$. This is the only one of the three that instantiates the strong claim of §5 under the §11 criterion.

### 14.3 Benchmark suite (existing, real, low-cost to run)

- **Multi-Query Associative Recall / MQAR** (Arora, Eyuboglu, Timalsina, Johnson, Poli, Zou, Rudra & Ré, "Zoology," ICLR 2024; arXiv:2312.04927). A sequence contains several key–value pairs followed by queries for those keys; the model must retrieve the correct value for each. Difficulty is controllable via vocabulary size, number of pairs, and sequence length. This is close to a direct implementation of the "cue → reconstruction" test in §7 and Prediction 3, and is small enough to run on a single GPU.
- **POPGym** (Morad, Kortvelesy, Bettini, Liwicki & Prorok, ICLR 2023; arXiv:2303.01859). Fifteen partially observable RL environments (diagnostic memory tasks, noisy control, card-matching games) with more than a dozen published memory-architecture baselines already benchmarked, making it a ready-made testbed for comparing Architectures A/B/C under matched training budgets on genuine sequential decision tasks rather than only language-model perplexity.
- **Continual-learning split-task protocol.** Standard sequential-task-block training with no rehearsal, measuring backward transfer, for the interference/plasticity test in Prediction 4.
- **Reservoir-computing memory-capacity task** (Jaeger, 2001 protocol). The cheapest, fastest diagnostic — a single scalar per architecture — useful as a first-pass filter before committing to the more expensive MQAR/POPGym runs.

### 14.4 Minimal first experiment (weekend-scale, single GPU)

1. Build a small MQAR-style synthetic task: vocabulary ≈ 256 tokens, 8–32 key–value pairs, sequence length 128–256.
2. Train three models at matched parameter count (roughly 1–5M parameters): (a) an oracle exact-match hash-table baseline (not learned — an upper-bound sanity check, not a real comparison point); (b) a 2-layer GRU (Architecture B); (c) a small fast-weight layer implementing a delta-rule associative update (Architecture C, following Ba et al., 2016, or a minimal Titans-style test-time update).
3. Train all models with clean (uncorrupted) queries only.
4. At test time only, corrupt a controllable fraction of characters in the query key and plot accuracy vs. corruption percentage for (b) and (c).
5. **Expected signature if the strong claim holds:** (c) should degrade more gracefully than (b) at moderate corruption levels, because a similarity-based associative update reconstructs approximately from a partial cue, whereas a fixed-recurrence model that has learned an effectively exact-match strategy should show a sharper accuracy cliff at the first corrupted character. If (b) and (c) degrade identically, that is evidence against the strong claim adding anything beyond what a well-trained weak-form system already achieves, and is exactly the kind of negative result the falsification stance in §16 calls for.

This experiment needs no infrastructure beyond a standard PyTorch training loop and is genuinely runnable in a few days.

### 14.5 Hardware-substrate parallel

At the physical level, Architecture C is the algorithmic analog of in-memory computing (PCM, RRAM, memristive crossbars), where a memory cell's physical conductance state directly participates in a multiply-accumulate operation rather than being fetched across a bus. This is an analogy across levels of description, not a proof that one requires the other: Titans-style test-time memory runs perfectly well on a GPU, and a memristive crossbar can still be used as nothing more than a fast literal store if it is read out through fixed addressing. The genuinely separate, substrate-level claim worth stating explicitly (Prediction 6, §13) is that the *efficiency* advantage of strong-form memory should be disproportionately larger on substrates where memory and computation are physically co-located than on von Neumann hardware, where the cost of moving data is already paid regardless of algorithmic structure. This is a distinct, later-stage experiment from the algorithmic one in §14.4, and should not be conflated with it.

### 14.6 Relationship to CAF

This program bears directly on two already-open CAF questions: what minimal persistent state is required for general adaptive intelligence, and how `EpisodicMemory` should be designed. As currently described, `EpisodicMemory`'s priority-eviction-and-consolidation design is architecturally closer to Architecture A/B (a store with consolidation, read out into a fixed transition function) than to Architecture C. The §11 criterion and the §14.4 experiment give a concrete, cheap way to decide — on CAF's own world-model backbone (`VariationalWorldModel` / `StructuralCausalModel`), before committing engineering effort to the memristive/PCM hardware line — whether making `EpisodicMemory`'s state directly parametrize the transition function (rather than being consulted by it) is worth the added complexity. The algorithmic question is answerable in days; the hardware-substrate question is not. Running the cheap experiment first is the practical order of operations this framework recommends.

---

## 15. Scientific Status

**Established, and now explicitly attributed (§3):** belief-state/sufficient-statistic theory for partial observability; recurrent/state-space models as a formalization of "state changes future computation"; reconstructive (non-literal) retrieval; attractor-based associative memory with quantitative capacity bounds; multi-timescale memory as neuroscientifically grounded; a working quantitative memory-capacity metric.

**Partially addressed by this revision, not fully settled:** the "is this just state" objection now has a proposed additional criterion (timescale separation, plus parametrizing-vs-input-to $F$), which sharpens the definition but has not been experimentally validated as the *right* additional criterion — it is itself a hypothesis, testable by the same program in §14.

**Still open:** whether strong-form memory (§5) produces categorically different, not just quantitatively different, computational behavior at matched compute; what mathematical quantity best measures a memory system's future computational value across architectures; whether the substrate-locality prediction (§13, Prediction 6) holds once actually measured on non-von-Neumann hardware rather than argued analogically.

---

## 16. Research Philosophy

$$
\boxed{\text{Do not protect the hypothesis. Try to destroy it.}}
$$

If a well-tuned weak-form architecture (Architecture B, or Architecture A with a good index) matches Architecture C's graceful-degradation and interference behavior at equal compute, the strong claim in §5 is not supported, and the paper's contribution reduces to the terminology and the related-work synthesis in §3  which would still be useful, but would not license the stronger architectural claims in §14.6. The experimental program in §14 is designed so that this negative outcome is a clean, visible result, not something that can be quietly reinterpreted as a partial success.

---

## 17. Final Open Question

$$
\boxed{\textbf{Is memory the archive of an intelligent system, or is memory the mechanism by which the system becomes a different computational system through experience — and if the latter, does that difference show up in anything measurable beyond what a well-designed state-space model already provides?}}
$$

The second clause is the addition this revision insists on: the original question is compelling but untestable as posed. Section 14 exists to make it testable.
