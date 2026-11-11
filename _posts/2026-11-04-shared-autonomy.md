---
layout: distill
title: "Shared Autonomy: Aligning Robot Assistance with Human Intent"
description: "How should a robot help without fighting its user? We study intent-aware assistance in high-speed racing and belief-conditioned latent actions for assistive manipulation, comparing their methods, evidence, and limitations."
date: 2026-11-04
future: true
htmlwidgets: true
ready: true 
authors:
  - name: "Vu-Anh Le"
  - name: "Jameson Sandler"
  - name: "Anqi Li"
  - name: "Sudeesh Murugesan"
bibliography: 2026-11-04-shared-autonomy.bib
toc:
  - name: "Introduction"
  - name: "Shared Autonomy and Its Challenges"
  - name: "Dreaming to Assist"
  - name: "Shared Autonomy with Learned Latent Actions"
  - name: "Comparison and Key Takeaways"
  - name: "Open Questions and Discussion"
  - name: "References"
---

## Introduction

A robot can execute movements that are difficult for a person to specify directly, yet the person usually understands *what* they want better than the machine does. **Shared autonomy** aims to combine these complementary strengths: a human supplies goals, preferences, and corrective inputs, while an autonomous controller helps translate those inputs into successful actions.

The challenge is not simply to maximize the amount of automation. A six-degree-of-freedom robot arm is awkward to operate with a low-dimensional joystick, especially when a task requires both reaching for an object and manipulating it precisely. A racing driver, meanwhile, may know whether they intend to overtake a car but lack the reaction time to steer optimally through a difficult corner. In either setting, a technically competent intervention can become counterproductive if it conflicts with the person's actual objective.

This lecture examined two complementary approaches. **Dream2Assist** predicts a driver's discrete tactical intent and learns corrective actions that advance that intent <d-cite key="decastro2024dreaming"></d-cite>. **Latent Actions + Shared Autonomy (LA+SA)** instead learns a low-dimensional control interface whose meaning changes with the robot's belief about the user's goal <d-cite key="jeon2020shared"></d-cite>. Together they motivate a central question: **How can assistance become more capable while preserving the user's ability to express, revise, and ultimately control their own objective?**

{% include figure.liquid path="assets/img/2026-11-04-shared-autonomy/shared-autonomy-overview.svg" class="img-fluid rounded z-depth-1" alt="Diagram of human and robot inputs combining into shared control and the executed action." caption="Figure 1: Shared autonomy combines human input with machine assistance. Both the inferred objective and the strength of assistance affect whether the final action serves the user. Original schematic based on the lecture's motivating examples." %}

## Shared Autonomy and Its Challenges

Shared autonomy lies between pure teleoperation, in which the human commands the machine, and full autonomy, in which the machine selects and executes actions independently. One common implementation blends a human command $a_h$ with a robot-proposed command $a_r$:

$$
a=(1-\alpha)a_h+\alpha a_r,\qquad 0\leq\alpha\leq1.
$$

Here, $\alpha=0$ corresponds to direct human control and $\alpha=1$ to full robot control. This convex blend is the formulation used by LA+SA; Dream2Assist uses a related but distinct **add-and-clip** rule for steering and acceleration. In either case, determining an appropriate robot action requires more than deciding how much control to assign to the machine.

Two problems recur throughout the lecture. First, **intent is partially observed**: a robot sees joystick motions or vehicle controls, not the human's underlying plan. Second, **the control interface is constrained**: the human's input may have far fewer degrees of freedom than the robot, and small mistakes can be amplified during high-speed or fine-motor tasks. The methods below respond to different parts of this difficulty—one learns *which correction to make*, and the other learns *what a human command should mean*.

## Dreaming to Assist

*DeCastro et al., “Dreaming to Assist: Learning to Align with Human Objectives for Shared Control in High-Speed Racing” (CoRL 2024).* <d-cite key="decastro2024dreaming"></d-cite>

### Motivation and problem formulation

In high-speed racing, the assistant must support tactical decisions before there is time for explicit communication. A driver might try to **pass** an opponent, **stay behind**, or pass from the **left** rather than the **right**. These choices can demand conflicting control actions: slowing down is helpful for staying behind but may frustrate an attempted overtake.

The paper formulates control as a two-player partially observable decision process. The assistant observes features of the track, the ego and opponent vehicles, and the human's steering and acceleration. The human's objective $y$ is latent. Both controllers output normalized continuous actions $a_t^H,a_t^A\in[-1,1]^2$; the vehicle receives their coordinate-wise sum, clipped to its admissible control range:

$$
a_t^{\mathrm{exec}}=\mathrm{clip}(a_t^H+a_t^A,-1,1).
$$

Thus, success means more than making the vehicle fast or safe in isolation. The intervention should improve task performance **while remaining consistent with the driver's intended maneuver**.

### Method: an intent-aware world model

Dream2Assist builds on *Dreamer*, a model-based reinforcement learning approach that learns a recurrent state-space model (RSSM) and trains policies using imagined trajectories <d-cite key="hafner2019learning"></d-cite>. The RSSM combines a deterministic recurrent state $h_t$ (memory of recent interactions) with a stochastic latent state $z_t$ (uncertainty about the environment). Observation, reward, and continuation heads predict what the world might do next. Dream2Assist adds an **intent-prediction head** $p_\phi(y_t\mid h_t,z_t)$, training the representation to encode not only vehicle dynamics but also the objective underlying the observed human actions.

The training pipeline has three stages:

**Stage 1.** (_Construct diverse synthetic drivers_) Train separate policies for passing, staying behind, left-biased driving, and right-biased driving. Save intermediate checkpoints to represent varying levels of skill, rather than only expert behavior.


**Stage 2.** (_Learn an assistant across this population_) Train its RSSM to predict dynamics, rewards, and discrete intent; train an actor–critic policy on model-generated rollouts. This is a form of *fictitious co-play*: the assistant learns to work with many plausible human partners.

**Stage 3.** (_Deploy the assistant online_) Infer the driver's current objective from their behavior and apply continuous steering and throttle corrections appropriate to that objective.

{% include figure.liquid path="assets/img/2026-11-04-shared-autonomy/dream2assist-pipeline.svg" class="img-fluid rounded z-depth-1" alt="Dream2Assist pipeline from synthetic driver behavior through an intent-aware recurrent state-space model to the assistant policy." caption="Figure 2: Conceptual Dream2Assist pipeline. Objective-labeled synthetic driver populations support an RSSM that jointly models the driving scene and human intent; the assistant uses that representation to select corrective actions. Original schematic after DeCastro et al. (2024) and lecture slides 22–26." %}

The assistant's reward incorporates three competing considerations: common driving objectives (finishing, avoiding collisions, staying on track), a penalty on large interventions, and alignment with the inferred human objective. The intervention term is

$$
r_t^{\mathrm{interv}}=-\|a_t^A\|_2.
$$

Importantly, the model can evaluate a proposed action using the predicted reward of an expert driver with the inferred intent. The paper also explores an expert-action matching term, but the results discussed in the lecture use expert-reward alignment, since direct expert-action matching performed worse. This distinction helps explain why correctly *classifying* intent need not be enough: the assistant must also model the consequences of its actions.

### Experiments and results

Evaluation was conducted in the CARLA simulator on two sections of Thunderhill Raceway: a relatively easy 370 m straightaway and a harder 960 m hairpin section. The experiments separately contrasted *pass vs. stay-behind* and *left vs. right* preferences, averaging outcomes over five imperfect synthetic drivers. Baselines included ordinary Dreamer, which lacks intent-aware reward alignment, and Dreamer-AIL, an adversarial imitation-learning assistant without the same RSSM-based intent-aware planning structure.

The evaluation tracked change relative to unassisted driving in track progress, human-objective return, and collisions. The most informative findings were:

- (_In difficult hairpin settings, Dream2Assist improved track progress by roughly 27–72 m in the reported conditions_), without increasing collisions, while also better satisfying the simulated driver's preferred maneuver.
- (_Skill level mattered._) Imperfect drivers generally benefited most. More capable drivers often needed smaller corrections, and extra intervention could occasionally reduce their performance.
- (Generic assistance could optimize the wrong behavior.] Dreamer sometimes favored staying behind an opponent even when the driver wished to pass, while Dreamer-AIL could freeze, collapse toward a single maneuver, or drive too aggressively.
- (_Intent accuracy did not guarantee useful help._) Intent classification achieved approximately **0.95–1.00 F1** in the lecture's reported tests. A non-RSSM ablation could classify intent nearly as well, yet assisted less effectively—suggesting the importance of learned dynamics and reward predictions.

A more demanding test swapped synthetic drivers mid-race to simulate a sudden change in intent. Classification performance initially fell, then recovered to high accuracy after approximately **4–8 seconds**. This shows some adaptability, but also exposes a potentially significant lag in a high-speed setting.

### Limitations

The evaluated 2024 system was trained and tested with synthetic human policies, not real drivers sharing control in the reported experiments. Its set of tactical intents was small and predefined, and the opponent vehicles replayed recorded trajectories rather than adapting to the driver. The approach also depended on expert models for each known objective. These choices make the method controlled and interpretable, but leave open questions about natural human behavior, novel objectives, multi-agent interactions, and the safety of assistance during intent transitions.

## Shared Autonomy with Learned Latent Actions

*Jeon, Losey, and Sadigh, “Shared Autonomy with Learned Latent Actions” (RSS 2020).* <d-cite key="jeon2020shared"></d-cite>

### Motivation: reaching is not manipulating

Assistive feeding illustrates a different limitation of conventional shared control. A user must first move a robotic arm toward an item, then perform a precise manipulation—cutting tofu, stabbing a marshmallow, or scooping through icing. Simple goal-directed assistance is useful during the first phase but insufficient during the second. At the same time, asking someone to manipulate every degree of freedom of a robot arm with a two-dimensional joystick is cumbersome.

The key insight is that **the meaning of a low-dimensional input should depend on context**. Early in a task, pushing a joystick can communicate *which object to approach*. Once the robot is confident about the object, that same joystick movement can instead express *how to manipulate it*. The transition is learned through a context-conditioned latent-action decoder, rather than through explicit manual switching between control modes.

### Method: belief-conditioned decoding and shared guidance

Let $s$ denote the robot configuration, $z\in\mathbb{R}^d$ the joystick input, and $a\in\mathbb{R}^m$ the robot action, with $d<m$. Let $\mathcal{G}$ contain candidate discrete goals, and let $b(g)$ be the robot's belief that $g$ is the intended goal. The learned decoder converts low-dimensional input into a high-dimensional action:

$$
a_h=\phi(z,s,b).
$$

The robot independently proposes goal-weighted guidance, then blends that guidance with the human's decoded command:

$$
a_r=\sum_{g\in\mathcal{G}}b(g)(g-s),\qquad
a=(1-\alpha)\phi(z,s,b)+\alpha a_r.
$$

The decoder $\phi$ is trained offline on demonstrated trajectories containing states, actions, and goal beliefs. Online, each joystick input is decoded using the current state and belief; the controller then computes assistance, executes the blended action, and updates its belief based on the user's behavior.

{% include figure.liquid path="assets/img/2026-11-04-shared-autonomy/latent-actions-pipeline.svg" class="img-fluid rounded z-depth-1" alt="Learned latent joystick decoding conditioned on robot state and goal belief, combined with goal-weighted autonomy." caption="Figure 3: In LA+SA, the belief-conditioned decoder translates the same low-dimensional joystick signal into goal-directed motion or precise manipulation. A separate goal-weighted assistant guides the robot toward likely targets. Original schematic after Jeon et al. (2020) and lecture slides 35–38." %}

### Interactive intuition: confidence and assistance

The visualization below illustrates how the *belief over goals* and the *assistance weight* play different roles. Increasing $b(g_1)$ redirects the goal-weighted autonomous suggestion toward goal 1; increasing $\alpha$ gives that suggestion more influence in the executed command. The human-action arrow is held fixed in this simplified demonstration—it is **not** a reproduction of the paper's learned decoder, whose behavior also changes with belief.

<iframe src="{{ site.baseurl }}/assets/html/2026-11-04-shared-autonomy/latent-control-demo.html" title="Illustrative shared autonomy belief and blending demo" width="100%" height="455" loading="lazy" style="border:1px solid #dde4ec;border-radius:12px;"></iframe>

### Why train for goal accessibility?

There is an important failure mode: once a robot becomes confident in the **wrong** target, both its autonomous guidance and a poorly trained latent space may reinforce that mistake. If the decoder provides no actions that move toward an alternative goal, the user cannot correct the robot merely by moving the joystick.

The paper studies this trade-off using the goal error $e(t)=g^{\ast}-s(t)$ and a Lyapunov function $V(t)=\tfrac12\|e(t)\|^2$. Under the paper's stated bounded-control assumptions, a sufficient condition for decreasing error is

$$
b(g^{\ast})\|e\|\gt\sigma_h+\big(1-b(g^{\ast})\big)\sigma_r,
$$

where $\sigma_h$ bounds the human's decoded command and $\sigma_r$ bounds the influence of competing goals. Greater confidence in the correct goal tightens the region toward which the robot converges; confidence in an incorrect goal can instead make switching difficult. This is a **sufficient condition for local progress**, not an unconditional global convergence guarantee.

To make alternative goals reachable, the authors add an entropy term during *offline decoder training*:

$$
\mathcal{L}=\|a_{\mathrm{demo}}-\hat a\|_2^2
+\lambda\sum_{g\in\mathcal{G}}p_{(s,b)}(g)\log p_{(s,b)}(g).
$$

Here, $p_{(s,b)}(g)$ measures the relative accessibility of each goal through the available latent actions; it is **not the same quantity as** the posterior goal belief $b(g)$. Since $\sum_g p(g)\log p(g)=-H(p)$, minimizing the loss rewards a decoder that reconstructs demonstrations **and** preserves multiple possible goal-directed motions. In simulations, this design helps a user undo an earlier mistake or switch to a newly preferred target.

### Simulations and user study

The simulation studies examined imperfect teleoperators, changing goals, people learning a control interface during a task, and targets not seen during training. LA+SA generally reduced reaching error and completion time, especially for noisy users. **LA+SA with entropy** was more resilient when a simulated user changed goals after the robot had developed a confident but incorrect belief. New targets were handled by mapping the unfamiliar context to an equivalent familiar one and transforming the decoded action back, rather than retraining the decoder.

The physical-robot study recruited **10 able-bodied participants**, each using a **2-DoF joystick** to operate a **7-DoF robot arm** in two eating tasks (tofu and a multi-step dessert). The experiment compared four conditions: direct retargeting (R), retargeting plus shared autonomy (R+SA), latent actions alone (LA), and the combined LA+SA method.

Results supported the complementarity of the two components:

- **Shared autonomy improved goal accuracy** across tasks, with a reported main effect of $F(1,18)=29.9$ and $p<0.001$.
- **Latent actions enabled changes in fine-grained manipulation preferences.** Participants could move between stabbing, scooping, and dipping motions rather than remaining in one default manipulation pattern.
- **The combination was most efficient:** LA+SA produced the lowest total and idle times among the tested conditions, with reported pairwise comparisons at $p<0.05$.

These results provide human-participant evidence that shared guidance and a learned control interface solve distinct problems. Nevertheless, the study was small, recruited only able-bodied participants, and used a restricted family of feeding tasks and demonstrated motions; deployment to the broader population of assistive-robot users would require additional evaluation.

## Comparison and Key Takeaways

Although these systems operate in very different domains, their successes and failures point to a common design principle: **automation should help realize the human's objective without removing the human's ability to choose or change it**.

| Dimension | Dream2Assist | Latent Actions + Shared Autonomy |
|:--|:--|:--|
| **Domain** | High-speed driving | Assistive robot manipulation |
| **Main uncertainty** | Which discrete maneuver the driver intends | Which goal the user wants, and which continuous manipulation they prefer |
| **Learned mechanism** | Intent-aware RSSM plus assistive policy | Context-conditioned low-dimensional action decoder |
| **How assistance acts** | Steering/throttle corrections added to human control | Goal-weighted guidance blended with decoded joystick commands |
| **Characteristic failure** | Wrong maneuver inferred or excessive corrections | Incorrect goal belief makes alternatives unreachable |
| **Key response** | Intent-conditioned expert reward and intervention penalty | Belief-conditioned decoding and entropy-based goal accessibility |
| **Core evidence in lecture** | Synthetic-driver CARLA experiments | Simulations and a 10-participant eating study |

Three lessons emerge. **First**, the assistant should represent *why* a person acts, not merely imitate the resulting action: two visually similar driving situations may require opposite interventions. **Second**, confidence must be coupled with *reversibility*: even a very confident model must leave a practical path for the human to correct it. **Third**, greater assistance is not automatically better. The appropriate intervention depends on task difficulty, user skill, uncertainty, and the cost of suppressing alternatives.

## Open Questions and Discussion

The lecture concluded with questions about when assistance becomes over-assistance. Several are especially important for designing future systems:

1. If a driver signals an overtake that appears unsafe, should the assistant optimize for the intended maneuver, delay it, or override it? What should trigger that decision?
2. Dream2Assist can take several seconds to recover after a simulated intent switch. How should an assistant behave while its estimate is unreliable, especially in a fast-changing environment?
3. A latent action may coordinate many robot joints at once but be hard for a human to interpret. Could online adaptation, feedback, or more interpretable constraints make the interface easier to learn?
4. Should we measure intervention magnitude, goal error, successful human corrections, subjective agency, or all of them? How would these metrics change across users and tasks?
5. How would the systems scale to unfamiliar intents, reactive opponents, new objects, or users whose control patterns vary substantially from training data?

A natural direction is to combine the two ideas: use a predictive model to maintain uncertainty over human objectives **and** a latent control interface designed to preserve controllable escape routes when that uncertainty is wrong. Such a system would not treat confidence as permission to eliminate alternatives; it would learn to assist while remaining corrigible to the user's next action.

## References

- DeCastro et al. (2024), [*Dreaming to Assist: Learning to Align with Human Objectives for Shared Control in High-Speed Racing*](https://arxiv.org/abs/2410.10062). CoRL 2024.
- Jeon, Losey, and Sadigh (2020), [*Shared Autonomy with Learned Latent Actions*](https://www.roboticsproceedings.org/rss16/p011.html). RSS 2020.
- Hafner et al. (2019), [*Learning Latent Dynamics for Planning from Pixels*](https://arxiv.org/abs/1811.04551). ICML 2019.
