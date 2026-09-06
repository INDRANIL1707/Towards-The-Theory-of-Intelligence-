# Paper I — Viability-Constrained Epistemic Agency: A Formal Theory of Endogenous Goal-Directed Intelligence

**Status:** Research hypothesis with partial proofs. Sections are explicitly labeled Theorem, Proposition, Conjecture, or Assumption — this labeling is load-bearing, not decorative, and is maintained throughout.

---

## Abstract

We formalize a narrow but rigorous claim about the origin of goal-directed behavior in an artificial agent that receives no externally specified task reward. The agent is embedded in a partially observable stochastic environment, maintains a Bayesian belief over hidden state and hidden causal structure, and is constrained by a hard viability filter that excludes any policy failing to keep a set of physical resource variables inside a survivable region. Subject to this filter, the agent selects actions to maximize a weighted combination of causal information gain and learning progress. We prove that (1) any admissible action's expected information gain is non-negative (a restatement of a standard mutual-information identity, included for completeness), (2) under a stated equal-cost assumption, an agent with positive epistemic weight strictly prefers at least one non-null action over inaction, and (3) no such preference can be derived from viability and the environment model alone — an explicit value primitive is mathematically necessary. We then state, without proof, the central open conjecture this program exists to test: that this coupling of viability, causal curiosity, and learning progress is *sufficient* to produce open-ended competence growth. We identify the specific mathematical gaps between what is proven and what is conjectured, and we situate every component against its true precedent in the literature, since several of the individual mechanisms substantially predate this synthesis and should be credited as such.

---

## 1. Introduction and Scope

This paper does not claim to prove that general intelligence exists as a derivable mathematical consequence of any set of axioms. No such proof is known to exist, and there are principled reasons — discussed in Section 9 — to expect that none can, in the fully general case. What can be done, and what this paper does, is narrower and more useful: take one specific candidate mechanism for the *origin* of goal-directed exploratory behavior in an agent with no external reward, state it with full mathematical precision, prove what actually follows from the stated assumptions, and identify exactly where proof gives way to hypothesis.

The organizing question is:

> Can an agent with a hard-coded survival constraint and a purely epistemic objective (reducing uncertainty about the causal structure of its world) be shown, mathematically, to prefer informative action over inaction — and if so, under what assumptions does that preference survive contact with realistic costs?

Everything that follows either answers this precisely or states plainly that it does not yet.

---

## 2. Formal Environment

The environment is a partially observable stochastic dynamical system

$$\mathcal{E} = (\mathcal{S}, \mathcal{A}, \mathcal{O}, T, \Omega, \rho)$$

with hidden state transitions $s_{t+1} \sim T(\cdot \mid s_t, a_t)$ and observations $o_t \sim \Omega(\cdot \mid s_t)$. The agent does not observe $s_t$; it maintains a belief

$$b_t(s, \theta, M) = P(s_t = s, \theta, M \mid h_t)$$

over the physical state $s$, unknown parameters $\theta$, and a discrete or structured variable $M$ indexing candidate causal mechanisms (e.g., which of several structural hypotheses generates the observed transitions). This is a standard POMDP augmented with model uncertainty over $(\theta, M)$ jointly — the formulation is not new; it is the same object used throughout Bayes-adaptive reinforcement learning (Duff, 2002; Ghavamzadeh et al., 2015) and Bayesian experimental design in sequential settings (Chaloner & Verdinelli, 1995). Framing the agent's core question as "what could the state and mechanism be, given what I've observed" rather than "what is the state" is the correct emphasis for what follows, but it is an emphasis, not a new formal object.

---

## 3. Viability as a Hard Constraint

**Definition 1 (Viability variable and admissible region).** Let $v_t \in \mathcal{V}$ denote a physical resource vector (energy, structural integrity, compute budget, memory integrity) and $K \subseteq \mathcal{V}$ an admissible region. The finite-horizon survival probability under policy $\pi$ from belief $b$ is

$$V_H(b, \pi) = P_\pi(v_{t:t+H} \in K \mid b_t = b).$$

**Definition 2 (Admissibility).** A policy is admissible iff $V_H(b, \pi) \ge 1 - \epsilon$.

This is not a novel mathematical structure. It is a direct application of viability theory (Aubin, 1991) as a hard action-space constraint, which is formally closest to constrained MDPs (Altman, 1999) and the safe-exploration reinforcement learning literature (García & Fernández, 2015). The substantive design decision — stated as **Assumption 1** below, not a theorem — is treating survival as a *constraint on the feasible policy set* $\Pi_{\text{safe}}$ rather than as one more additive term inside a scalar reward. This ordering matters and is worth defending explicitly: an additive safety term can always, in principle, be outbid by a sufficiently large value on some other term, whereas a hard constraint on the feasible set cannot be outbid by construction. We adopt the hard-constraint formulation and flag it as an assumption because its cost — analyzed in Section 8 — is that a hard constraint estimated from a wrong or incomplete model provides no guarantee about the real world, only about the agent's model of it.

---

## 4. Epistemic Objectives

**Definition 3 (Expected information gain).** For candidate action $a$ and predicted observation $O'$,

$$IG(a) = I(\Theta ; O' \mid h_t, a) = H(\Theta \mid h_t) - \mathbb{E}_{o'}\left[H(\Theta \mid h_t, a, o')\right].$$

This is Lindley's (1956) expected information gain criterion for optimal experimental design, applied here to action selection rather than to fixed experimental designs — the criterion itself is not new; the application to online agent behavior is standard in Bayesian active learning and Bayesian experimental design for sequential decision-making (Chaloner & Verdinelli, 1995; Houlsby et al., 2011 for the modern deep-learning instantiation).

**Proposition 1 (Non-negativity).** $IG(a) \ge 0$ for all $a$, with equality iff $O'$ is conditionally independent of $\Theta$ given $(h_t, a)$.

*Proof.* This is the standard non-negativity of mutual information, $I(X;Y) = H(X) - H(X\mid Y) \ge 0$, which follows from Jensen's inequality applied to the concavity of $\log$ (Cover & Thomas, 2006, Theorem 2.6.3). No environment-specific assumption is required. $\blacksquare$

This proposition is included for completeness and correctness, not because it is a novel result — it is a textbook identity. Its only substantive role here is as a building block for Theorem 1 below.

### 4.1 Why raw prediction error is not the right quantity

**Proposition 2 (Prediction error is not information gain).** There exist environments and actions $a$ for which $\mathbb{E}[L(O', \hat{O}')]$ is large while $IG(a) \approx 0$.

*Proof (by construction).* Let $O_t' = \eta_t$ where $\eta_t$ is i.i.d. noise independent of $\Theta$ and of $h_t$. Then $\mathbb{E}[(O' - \hat{O}')^2]$ can be made arbitrarily large by increasing the variance of $\eta_t$, while $I(\Theta; O' \mid h_t, a) = 0$ exactly, since $O'$ is constructed to be statistically independent of $\Theta$. $\blacksquare$

This is a deliberately simple construction, but it is the formal core of a real and well-documented failure mode in prediction-error-based curiosity: agents that maximize prediction error alone are provably attracted to unlearnable stochastic noise sources (the "noisy TV problem," documented empirically in Burda et al., 2018, "Large-Scale Study of Curiosity-Driven Learning," and analyzed further in Pathak et al.'s original curiosity-driven exploration work, 2017, which this program should credit directly rather than treat as background). Proposition 2 gives that empirical observation a one-line proof: prediction error and mutual information are different functionals of the same quantity, and they can be made to disagree arbitrarily by construction. This is why $IG$, not raw prediction error, is the correct primitive — and it is why the earlier informal claim "$PE \ne IG$" deserves to be a proposition, not an assertion.

### 4.2 Causal information gain

**Definition 4 (Causal information gain).** Let $\Theta_C$ parameterize the space of causal hypotheses (e.g., competing structural causal models $M_1, \ldots, M_n$). For an intervention $do(a)$,

$$CIG(a) = I(\Theta_C ; O' \mid h_t, do(a)).$$

This is not a new information-theoretic quantity distinct from Definition 3; it is $IG$ evaluated specifically under $do(a)$ rather than under passive conditioning, restricted to the causal-structure component of $\Theta$. Its correct precedent is optimal experimental design for causal structure learning — specifically Hauser & Bühlmann (2012), who derive the minimum number of interventions required to fully identify a causal DAG in the worst case, and Hyttinen, Eberhardt & Hoyer (2013), who address the online, sequential intervention-selection version of the same problem. **This is the load-bearing precedent for the causal half of this framework, and it should be cited as such rather than presented as a new construction** — the genuine contribution here is not the quantity but its embedding inside a persistent, viability-constrained agent loop rather than a one-shot experimental design problem.

**Proposition 3 (Discriminative power).** If $P(x \mid do(a), M_1) \ne P(x \mid do(a), M_2)$ for some observable $x$, then $CIG(a) > 0$ with respect to the two-hypothesis subspace $\{M_1, M_2\} \subset \Theta_C$.

*Proof.* Direct consequence of Proposition 1 applied to the restricted hypothesis space $\{M_1, M_2\}$: if the interventional distributions differ, $O'$ is not conditionally independent of the hypothesis identity given $do(a)$, so mutual information is strictly positive. $\blacksquare$

### 4.3 Learning progress

**Definition 5 (Learning progress).** $LP(a) = \mathbb{E}[\mathcal{L}_t - \mathcal{L}_{t+1} \mid a]$, where $\mathcal{L}$ is a predictive loss.

This is Schmidhuber's compression-progress principle (1991, formalized further in Schmidhuber 2010, "Formal Theory of Creativity, Fun, and Intrinsic Motivation") — interestingness as the *derivative* of model quality, not its absolute level. It is included here specifically because $IG$ and $CIG$ alone do not exclude permanently unlearnable stochastic environments (Proposition 2 shows $IG \to 0$ for pure noise, but a partially structured, partially chaotic environment can sustain nonzero $IG$ indefinitely without ever converging). $LP$ is the correct additional filter for that intermediate case, and its precedent should be credited by name.

---

## 5. The Central Theorems

**Definition 6 (Objective).** For policy $\pi$,

$$J(\pi) = G(\pi) + \lambda_I IG(\pi) + \lambda_C CIG(\pi) + \lambda_L LP(\pi) + \lambda_E \mathcal{E}(\pi) - \lambda_R R(\pi) - \lambda_K C_{\text{compute}}(\pi)$$

subject to $\pi \in \Pi_{\text{safe}} = \{\pi : V_H(b_t, \pi) \ge 1-\epsilon\}$, and $\pi^* = \arg\max_{\pi \in \Pi_{\text{safe}}} J(\pi)$.

**Theorem 1 (Expected uncertainty reduction).** For any admissible action $a$ selected with $IG(a) > 0$, $\mathbb{E}[H(\Theta \mid h_t, a, O')] \le H(\Theta \mid h_t)$.

*Proof.* Immediate rearrangement of Proposition 1's identity. $\blacksquare$

**Limitation, stated explicitly:** this theorem is about the agent's *posterior entropy over its own model class* $\Theta$. It says nothing about whether $\Theta$ contains the true generative process. An agent can reduce $H(\Theta \mid h_t)$ monotonically while converging confidently on a wrong model, if the true process lies outside the agent's hypothesis space entirely (a standard model-misspecification failure — see Grünwald & van Ommen, 2017, on the specific pathologies of Bayesian updating under misspecification). This theorem proves convergence of belief, not correctness of belief, and conflating the two is a mistake worth naming explicitly rather than leaving implicit.

**Theorem 2 (Non-inertial exploration under equal cost).** Let $a_0$ be the null action. Suppose there exists an admissible $a^*$ with $IG(a^*) > 0$ (or $CIG(a^*) > 0$), $\lambda_I > 0$ (or $\lambda_C > 0$), and $G(a^*) = G(a_0)$, $R(a^*) = R(a_0)$, $C_{\text{compute}}(a^*) = C_{\text{compute}}(a_0)$. Then $J(a^*) > J(a_0)$, and hence $a_0 \notin \arg\max J$.

*Proof.* Direct substitution: $J(a^*) - J(a_0) = \lambda_I IG(a^*) > 0$ under the stated equalities. $\blacksquare$

**This proof is correct but the theorem is considerably weaker than it is often read as being, and this must be stated as part of the theorem, not as a caveat appended afterward.** The equal-cost assumption is not a simplifying convenience — it is the entire content of the result. In essentially every real embodied setting, exploration strictly dominates the null action in cost: it consumes more energy, incurs more physical risk, and often more compute (simulating candidate outcomes before acting). The theorem as stated proves existence of a preference for exploration in a knife-edge case; it does not prove robustness of that preference under a realistic cost gap $\Delta = R(a^*) - R(a_0) > 0$. The correct open question, restated formally, is:

**Open Problem 1.** Characterize the set of environments and cost gaps $\Delta$ for which $\lambda_I IG(a^*) > \lambda_R \Delta$ holds for at least one admissible $a^* \ne a_0$, as a function of the environment's information density.

This is not answered in this paper. It is the first thing a follow-up experimental program (Section 10) should measure directly, because it is the actual empirical content behind the qualitative claim "curiosity produces exploration."

**Theorem 3 (Necessity of a value primitive — the impossibility result).** Suppose $IG(a) = CIG(a) = LP(a) = 0$ for every admissible $a$, and $G(a) = G(a_0)$, $R(a) = R(a_0)$, $C_{\text{compute}}(a) = C_{\text{compute}}(a_0)$ for every $a$. Then $J(a) = J(a_0)$ for every admissible $a$, and no non-null action is uniquely optimal.

*Proof.* Direct substitution into Definition 6; every term is equal across all $a$, so $J$ is constant on the admissible set. $\blacksquare$

**Corollary.** No selection of a non-null action can be derived from the environment model and the viability constraint alone; some nonzero weighting on $G$, $IG$, $CIG$, $LP$, or $\mathcal{E}$ is a mathematically necessary primitive, not a derived quantity.

This result is correct and is, in our assessment, the most important claim in this entire research program, but it is not new *in kind* — it is a clean, correctly-formalized instance of Hume's is-ought gap (1739) and its contemporary form in AI theory, Bostrom's orthogonality thesis (2012): that an arbitrarily capable optimization process does not, by virtue of its capability alone, come with any particular final goal. Theorem 3 is the special case of that general principle for this specific objective form. Presenting it as a discovery specific to this framework would overclaim; presenting it as a rigorous, minimal instantiation of a much older and more general result is the accurate framing, and arguably strengthens the paper, since it shows the finding is not an artifact of a particular formalization choice.

---

## 6. The Convexity Problem With the Objective's Form

**Proposition 4 (Scalarization limits reachable policies).** For any fixed weights $(\lambda_I, \lambda_C, \lambda_L, \lambda_E, \lambda_R, \lambda_K)$, the policy $\pi^* = \arg\max J(\pi)$ lies on the convex hull of the achievable value frontier in the space of $(G, IG, CIG, LP, \mathcal{E}, R, C_{\text{compute}})$-tuples.

*Proof sketch.* This is the standard result for linear scalarization in multi-objective optimization (Das & Dennis, 1997; Miettinen, 1999): a weighted sum objective is a supporting hyperplane of the achievable value set, and its maximizer over that set is necessarily a point where a hyperplane of that orientation is tangent to the frontier — which excludes any point on a non-convex (concave-inward) region of the frontier, regardless of how the weights are tuned. $\blacksquare$

**Consequence.** If the true trade-off between, say, causal information gain and empowerment is non-convex for some class of environments — plausible, since they are qualitatively different kinds of value with no reason to trade off linearly — there exist policies that are excellent on one axis and merely adequate on the other that **no setting of the weights in Definition 6 can ever select.** This is a structural limitation of the objective's functional form, not a tuning problem, and is stated here as an open design question rather than resolved: a lexicographic ordering (viability first, absolute; then a Pareto-front search over the remaining terms) is one standard alternative that avoids this limitation at the cost of a harder-to-analyze optimization problem, and is the recommended next formalization to test against Definition 6 empirically (Section 10).

---

## 7. Failure Mode Taxonomy, Formalized

The failure modes below are stated informally in prior working notes on this project; here each is given a specific mathematical trigger condition, which is what makes the taxonomy usable as a test suite rather than a list of concerns.

| Failure mode | Formal trigger | Mathematical remedy |
|---|---|---|
| Infinite novelty-seeking (noisy-TV problem) | $\exists a: IG(a) \gg 0$ under a misspecified $\Theta$ that includes unlearnable stochastic nuisance parameters | Proposition 2 shows $IG \to 0$ for true independence; failure occurs specifically when $\Theta$'s prior wrongly assigns hypothesis mass to nuisance noise. Remedy: restrict $\Theta$ to structural/mechanism variables only, excluding exogenous noise parameters from the information-gain calculation |
| Curiosity overriding safety | $\lambda_I IG(a)$ dominates $J$ for some $a \notin \Pi_{\text{safe}}$ | Not possible by construction under Definition 6, since $\Pi_{\text{safe}}$ is a hard constraint on the feasible set, not a term in $J$ — this is the formal payoff of Assumption 1 |
| Paralysis | $\Pi_{\text{safe}} = \{a_0\}$ for all $b_t$ in some reachable belief region | Requires $\epsilon$-relaxation or a risk-sensitive variant of Definition 2; unresolved here, flagged as Open Problem 2 |
| Wireheading | Agent's action space includes $a_w$ such that $a_w$ directly modifies the channel computing $J$ rather than the environment | Requires the $J$-evaluation process to be causally downstream of, and non-modifiable by, any action in $\mathcal{A}$ — a hard architectural constraint, not a term in the objective; addressed in Paper II §6 |
| Model hallucination under Theorem 1 | Convergent posterior over $\Theta$ with true process $\notin \text{supp}(\Theta)$ | Grünwald & van Ommen (2017)-style misspecification; requires model-class expansion mechanisms, not addressed by Theorem 1 alone |

---

## 8. The Critical Gap: Model-Relative vs. Real Safety

**Proposition 5 (Compounding risk under fixed per-step tolerance).** If a policy satisfies $V_H(b_t, \pi) \ge 1-\epsilon$ independently at each of $N$ sequential decision points with $\epsilon$ fixed and nonzero, the probability of at least one viability failure across the full deployment is bounded below by $1 - (1-\epsilon)^N$, which approaches 1 as $N \to \infty$ for any fixed $\epsilon > 0$.

*Proof.* Direct union bound over $N$ approximately independent per-decision failure events, each with probability at most $\epsilon$; $(1-\epsilon)^N \to 0$ as $N \to \infty$. $\blacksquare$

This is not a hypothetical concern — it is the same reason long-horizon safe exploration is treated as a genuinely open problem in the safe reinforcement learning literature (a survey treatment appears in García & Fernández, 2015) rather than something a per-step viability filter resolves by construction. **Definition 2, as stated, is a per-decision guarantee, not a deployment-lifetime guarantee**, and any engineering claim built on top of it (Paper II) must not silently upgrade one into the other.

The proposed robust-control fix — $\max_\pi \min_{P \in \mathcal{P}_t} J(\pi; P)$ over an ambiguity set of plausible environment models (Iyengar, 2005; Nilim & El Ghaoui, 2005, for the formal distributionally-robust MDP machinery) — narrows but does not close this gap, because $\mathcal{P}_t$ is itself estimated from the same data and model class the whole apparatus is meant to be robust against. This regress is not resolved in this paper; it is named as **Open Problem 3**.

---

## 9. Why a Complete Proof of "This Architecture Produces General Intelligence" Should Not Be Expected

It is worth stating directly why Section 1 scoped this paper as narrowly as it did. The No Free Lunch theorem (Wolpert & Macready, 1997) proves that no policy-selection procedure outperforms any other when averaged uniformly over the space of all possible environments; any claim of general competence necessarily depends on a non-uniform prior over which environments are "real," and that prior cannot itself be derived from the same theory it grounds. Separately, Theorem 3's impossibility result shows that no value or goal can be derived from computation and environment structure alone. Together, these mean: even a fully successful empirical validation of this framework (Section 10) would establish that *this specific coupling of viability and epistemic objectives produces open-ended competence in the tested environment class* — a strong and useful result — not that intelligence in general has been "solved" or reduced to a closed-form derivation. Papers that promise the latter should be read skeptically; this one does not.

---

## 10. What Would Constitute Evidence, and What Would Falsify This

**Falsification criteria**, restated with the specific measurable trigger for each:

- **F1.** $CIG$-driven exploration shows no measurable advantage over plain $IG$-driven exploration in discriminating competing causal hypotheses (Definition 4 vs. Definition 3, run head to head) — this would mean the one genuinely novel component of the framework (embedding intervention-selection inside a persistent agent loop) contributes nothing beyond restating Hauser & Bühlmann (2012) in a different setting.
- **F2.** Theorem 2's preference for exploration disappears once a realistic cost gap $\Delta > 0$ is introduced (Open Problem 1) — this would mean the "curiosity beats inertia" result is a knife-edge artifact, not a robust phenomenon.
- **F3.** The viability constraint (Definition 2) produces paralysis ($\Pi_{\text{safe}} = \{a_0\}$) in any environment with nontrivial risk, rather than merely in pathological edge cases.
- **F4.** A conventional model-based RL baseline with a simple novelty bonus matches the full objective's performance at substantially lower implementation complexity, on the same task suite.

A companion experimental note (available separately) specifies the minimal four-condition test — null baseline, $G$-only, $G{+}IG$, $G{+}CIG$ — needed to check F1 and F2 directly before any larger claim is pursued.

---

## References

Altman, E. (1999). *Constrained Markov Decision Processes.*
Aubin, J.-P. (1991). *Viability Theory.*
Bostrom, N. (2012). The superintelligent will: motivation and instrumental rationality in advanced artificial agents. *Minds and Machines.*
Burda, Y. et al. (2018). Large-scale study of curiosity-driven learning. *arXiv:1808.04355.*
Chaloner, K. & Verdinelli, I. (1995). Bayesian experimental design: a review. *Statistical Science.*
Cover, T. & Thomas, J. (2006). *Elements of Information Theory* (2nd ed.).
Das, I. & Dennis, J. (1997). A closer look at drawbacks of minimizing weighted sums of objectives. *Structural Optimization.*
Duff, M. (2002). *Optimal Learning: Computational Procedures for Bayes-Adaptive Markov Decision Processes* (PhD thesis).
García, J. & Fernández, F. (2015). A comprehensive survey on safe reinforcement learning. *Journal of Machine Learning Research.*
Ghavamzadeh, M. et al. (2015). Bayesian reinforcement learning: a survey. *Foundations and Trends in Machine Learning.*
Grünwald, P. & van Ommen, T. (2017). Inconsistency of Bayesian inference for misspecified linear models. *Bayesian Analysis.*
Hauser, A. & Bühlmann, P. (2012). Two optimal strategies for active learning of causal models from interventions. *International Journal of Approximate Reasoning.*
Houlsby, N. et al. (2011). Bayesian active learning for classification and preference learning. *arXiv:1112.5745.*
Hume, D. (1739). *A Treatise of Human Nature.*
Hyttinen, A., Eberhardt, F. & Hoyer, P. (2013). Experiment selection for causal discovery. *Journal of Machine Learning Research.*
Iyengar, G. (2005). Robust dynamic programming. *Mathematics of Operations Research.*
Klyubin, A., Polani, D. & Nehaniv, C. (2005). Empowerment: a universal agent-centric measure of control. *IEEE Congress on Evolutionary Computation.*
Lindley, D. (1956). On a measure of the information provided by an experiment. *Annals of Mathematical Statistics.*
Miettinen, K. (1999). *Nonlinear Multiobjective Optimization.*
Nilim, A. & El Ghaoui, L. (2005). Robust control of Markov decision processes with uncertain transition matrices. *Operations Research.*
Pathak, D. et al. (2017). Curiosity-driven exploration by self-supervised prediction. *ICML.*
Schmidhuber, J. (1991). Curious model-building control systems. *IEEE International Joint Conference on Neural Networks.*
Schmidhuber, J. (2010). Formal theory of creativity, fun, and intrinsic motivation. *IEEE Transactions on Autonomous Mental Development.*
Wolpert, D. & Macready, W. (1997). No free lunch theorems for optimization. *IEEE Transactions on Evolutionary Computation.*
