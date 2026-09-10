# Homeostasis, Allostasis and Degeneracy

**Wanting what is needed: degeneracy in the causal mechanisms of homeostatic and allostatic motivation**

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/aslansd/Homeostasis-Allostasis-Degeneracy/blob/main/Homeostasis_Allostasis_Degeneracy.ipynb)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Python 3.10+](https://img.shields.io/badge/python-3.10%2B-blue.svg)](https://www.python.org/)

A single self-contained notebook that evolves small artificial agents whose only
selection pressure is **staying alive**, and then asks what mechanism they
actually acquired.

Motivation in organisms is not supplied by an external reward signal. It derives
from the need to hold essential physiological variables inside a viability zone,
which is why water is worth something to a dehydrated animal and nothing to a
sated one. Grounding artificial agents in the same constraint is an attractive
route to autonomy, but it raises a question that is rarely asked: if motivation
is allowed to *emerge* rather than being engineered, how do we establish which
mechanism an agent ended up with?

**The short answer: you often can't, from behaviour alone.** Half of the viable
agents evolved here regulate their bodies as well as any interoceptive
controller and would pass any behavioural test administered in the environment
they evolved in — yet nothing in them represents a need. They are open-loop
rhythms that happen to work. They collapse the moment the metabolism changes.

---

## Quick start

Click the Colab badge above, or open `Homeostasis_Allostasis_Degeneracy.ipynb`
in Jupyter and run all cells.

**No installation is required.** The notebook uses only NumPy, SciPy, pandas,
Matplotlib and NetworkX, all of which ship with Colab. The actual-causation
machinery is implemented from scratch, so [PyPhi](https://github.com/wmayner/pyphi)
is *not* a dependency — it is needed only for the optional cross-check in
Section 8.2.

With the default settings (12 evolutions × 4 conditions, 500 generations,
population 80) the full sweep takes roughly 25 minutes on a free Colab CPU
runtime. Reduce `N_RUNS` or `GENERATIONS` in Section 10 for a quicker look.
Evolution is deterministic given the seeds, so the numbers below reproduce
exactly.

## The model

Each agent carries two **essential variables** $x(t) \in [0,1]^2$ that deplete
autonomously and are replenished only by acting on the world. It is viable while
$x(t) \in (0,1]^2$; reaching $x_i \le 0$ ends the life.

The **drive** is the squared distance from a setpoint,

$$D(x) = \sum_i w_i\,|x_i - x_i^{*}|^{2}, \qquad x^{*} = 0.75,$$

so over-consumption is as costly as deficit. The **motivational salience** of an
outcome $o$ that would change the body by $\Delta_o$ is the drive reduction it
affords,

$$m(o \mid x) = D(x) - D(x \oplus \Delta_o),$$

a function of *need*, not of the outcome: the same ingestive act is appetitive
under deficit and aversive under surfeit. Fitness is nothing but cumulative
viability,

$$f = \frac{1}{T}\sum_t \mathbb{1}[\text{alive at } t]\left(1 - \frac{D(x(t))}{D_{\max}}\right).$$

No reward is defined anywhere, for the agent or for the evolutionary algorithm.

**Allostasis** is regulation of the *expected future* deviation. Given a
predictive context $c$, the anticipatory drive
$D_A(x,c) = \mathbb{E}\left[\sum_k \gamma^k D(x_{t+k}) \mid x, c, \pi\right]$
is minimised at a context-shifted setpoint

$$x^{*}_{A}(c) = \arg\min_x D_A(x,c) = x^{*} + \Delta(c),$$

so paying a present homeostatic cost to avoid a larger future one is exactly
"wanting what is needed *before* it is needed".

### Agents and design

Agents are **Markov brains**: networks of genetically encoded deterministic
logic gates in which both the wiring and the individual node functions evolve.
Four hidden units, five sensors, two motors (stay / left / right / ingest).
They forage on a circular 24-cell track carrying two spatially separated
resource patches, each replenishing one essential variable — so viability
requires leaving a patch once sated and travelling to the other.

A 2×2 design crosses two factors:

| | **H** — stationary world | **A** — patch A closes for 26 steps, cued 10 steps ahead |
|---|---|---|
| **I** — interoceptive sensors report $x_i < \theta$ | H-I | A-I |
| **N** — those sensors clamped off | H-N | A-N |

Without interoception an agent has no access to its own needs and must regulate
open-loop. Under allostatic demand, surviving the closure near the setpoint
requires an anticipatory upward shift of the target level of $x_1$.

## Two new measures

**Need-conditioned causal profile.** Transitions are partitioned by
physiological state (sated vs. deficit) and a causal profile computed for each,
making explicit how causal contributions redistribute with need.

**Motivational Modulation Index.** A causal reading of motivated behaviour:
action selection whose *causal organisation* is reconfigured by need, rather
than merely correlated with it.

$$\mathrm{MMI} = \frac{1}{|U|}\sum_{U \in S \cup H}\Big|\bar{\alpha}_c(U \prec M \mid \text{deficit}) - \bar{\alpha}_c(U \prec M \mid \text{sated})\Big|$$

Here $\alpha_c(U \prec M)$ is the causal contribution of unit $U$ to the motor
state, obtained from the actual-causation framework and distributed over units
by Shapley value.

## Results

Of 48 evolved agents, 36 were **viable** (survived every life with $f > 0.90$):
12/12 in H-I, 10/12 in A-I, 7/12 in each N condition.

| Finding | Numbers |
|---|---|
| Allostasis is costly, and real | fitness 0.948 vs. 0.970 (*p* = 8.5×10⁻⁵); anticipatory overfill before closure *p* = 3.8×10⁻⁴; cue acquires causal power over motors only where informative, $\alpha_c$ = 0.129 vs. 0 (*p* = 0.002) |
| Interoception is **not** needed to regulate well | fitness 0.961 vs. 0.958 (*p* = 0.32); within the stationary condition, 0.9706 vs. 0.9690 (*p* = 0.23) |
| …but the mechanism is different | $A^S_1$ = 1.00 vs. 1.82 bits (*p* = 1.1×10⁻⁴) — agents without interoception generate the missing information endogenously and run internal clocks |
| **Only need-gated agents are ultrastable** | **44% vs. 0% survival under an unannounced metabolic challenge (*p* = 5.9×10⁻⁴)** |
| Every viable agent is computationally unique | 0 identical causal profiles among 153 within-condition pairs |
| Degeneracy grows with regulatory demand | mean $L_1$ profile distance 86.2 (A-I) vs. 38.4 (H-I); A vs. H *p* = 5.1×10⁻⁹ |
| Behaviour underdetermines mechanism | *r* = 0.13 between behavioural and causal-profile distance; among the 10% most behaviourally similar pairs the profile distance still spans 12.0–80.4 |

Degeneracy at three levels among viable agents:

| Condition | *n* | behavioural dist. | structural classes | $L_1$ mean ± sd | $L_1$ min | identical pairs |
|---|---|---|---|---|---|---|
| H-I | 12 | 0.65 | 12 | 38.4 ± 15.5 | 14.3 | 0 |
| H-N | 7 | 0.64 | 7 | 10.5 ± 4.6 | 1.4 | 0 |
| A-I | 10 | 0.61 | 10 | 86.2 ± 33.3 | 20.5 | 0 |
| A-N | 7 | 0.67 | 7 | 27.3 ± 9.7 | 9.2 | 0 |

The **metabolic challenge** is the decisive test: from *t* = 60 the depletion
rate of $x_1$ rises by 80% for the rest of the life. It cannot be absorbed by
repeating the same cycle, because the agent must now spend more time ingesting.
A rhythm tuned to one depletion rate cannot retune itself; a controller gated by
its own deficit signal lingers at the patch for as long as the deficit persists.
This is Ashby's test of ultrastability in its strongest form.

## What is in the notebook

| Section | Contents |
|---|---|
| 1 | Formal framework: drive, motivational salience, allostatic setpoint |
| 2 | Markov brains: genome, logic gates, TPM, connectome |
| 3 | The `HomeoTrack` environment and the 2×2 design |
| 4 | Artificial evolution on pure viability |
| 5 | Behavioural degeneracy and the perturbation tests |
| 6 | Structural degeneracy |
| 7 | Interoceptive autonomy $A^S_k$ |
| 8 | Actual causation: causal profiles, MMI, computational degeneracy |
| 8.2 | Optional validation against PyPhi |
| 9–11 | Analysis sweep, statistics, figures |
| 12 | Inspect a single agent: connectome, regulation trace, causal account |

## Validation against PyPhi

The actual-causation code is a from-scratch reimplementation, so it is checked
against the reference implementation. Setting `RUN_PYPHI_CHECK = True` in
Section 8.2 installs PyPhi 1.2.0, embeds each brain in a `pyphi.Network`, forms
the corresponding `pyphi.actual.Transition`, and compares $\alpha$ link by link
against `find_mip`, plus the maximal causal links against `find_causal_link`.

Agreement is exact: **10,923 individual links and 108 maximal causal links**
across twelve evolved agents, and 2,538 links over random brains. The
implementation also reproduces the canonical values of Albantakis et al. (2019):
COPY = 1 bit, AND with both inputs on = 1 bit split 0.5/0.5 by Shapley value,
OR(1,0) = log₂(4/3) = 0.415.

Two conventions matter and are easy to get wrong:

1. **Repertoires factorise.** Purview nodes (effect direction) and mechanism
   nodes (cause direction) are treated as conditionally independent "virtual
   elements", so an effect repertoire is a *product* over purview nodes and a
   cause repertoire a *normalised product* over mechanism nodes. Using the exact
   joint instead agrees for single-node purviews but not in general.
2. **PyPhi rounds** $\alpha$ to `config.PRECISION` (6 decimals), so agreement
   must be assessed at that tolerance rather than at machine epsilon.

## Citation

A manuscript based on this code is in preparation. In the meantime, please cite
the repository and the work it builds on:

- Hu Z, Cingiler O, Bohm C, Albantakis L (2025) From function to implementation:
  exploring degeneracy in evolved artificial agents. *Neural Computation*
  37(9):1677–1708. https://doi.org/10.1162/neco.a.19
- Albantakis L, Marshall W, Hoel E, Tononi G (2019) What caused what? A
  quantitative account of actual causation using dynamical causal networks.
  *Entropy* 21(5):459. https://doi.org/10.3390/e21050459
- Albantakis L (2021) Quantifying the autonomy of structurally diverse automata:
  a comparison of candidate measures. *Entropy* 23(11):1415.
  https://doi.org/10.3390/e23111415 —
  [autonomy toolbox](https://github.com/Albantakis/autonomy)
- Keramati M, Gutkin B (2014) Homeostatic reinforcement learning for integrating
  reward collection and physiological stability. *eLife* 3:e04811.
- Sterling P (2012) Allostasis: a model of predictive regulation.
  *Physiology & Behavior* 106(1):5–15.
- Hintze A et al. (2017) Markov brains: a technical introduction.
  arXiv:1709.05601.
- Ashby WR (1952) *Design for a Brain: The Origin of Adaptive Behaviour*.
  Chapman & Hall.

## License

MIT — see [LICENSE](LICENSE).
