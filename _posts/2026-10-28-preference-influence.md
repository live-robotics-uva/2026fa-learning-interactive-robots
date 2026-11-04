---
layout: distill
title: "Preference, Influence"
description: "Learning human preferences from highlights in reinforcement learning and intent from latent representations in multi-agent interactions."
date: 2026-10-28
future: true
htmlwidgets: true
ready: true

# Reporter team authors
authors:
  - name: "Huu Binh Ta"
  - name: "Duc Anh Nguyen"
  - name: "Tristan Scott"

bibliography: 2026-10-28-preference-influence.bib

toc:
  - name: "Introduction"
  - name: "POLITE: Preferences Combined with Highlights in Reinforcement Learning"
    subsections:
      - name: "POLITE Motivation"
      - name: "POLITE Related Works"
      - name: "POLITE Methodology"
      - name: "POLITE Experiments"
      - name: "POLITE Conclusion"
      - name: "POLITE Limitations"

  - name: "LILI: Learning Latent Representations to Influence Multi-Agent Interaction"
    subsections:
      - name: "LILI Motivation"
      - name: "LILI Related Works"
      - name: "LILI Methodology"
      - name: "LILI Experiments"
      - name: "LILI Conclusion"
      - name: "LILI Limitations"
  - name: "Overall Conclusion"
  - name: "Discussion Questions and In-Class Clarifications"
    subsections:
      - name: "Questions about POLITE"
      - name: "Questions about Learning and Influencing Latent Intent (LILI)"  
  
  - name: "References"
---
*All authors are affiliated with the University of Virginia.*

## Introduction

This lecture report covers the **Preference, Influence** session in *Learning for Interactive Robots (CS 6501, Fall 2026)* at the University of Virginia.

> **Topic Overview**: Learning human reward functions and preferences from pairwise comparisons, rankings, and active information gathering.

#### Incorporating Human Preference

Robots are often trained with data-driven methods that require large datasets. Instead, a robot can learn directly from human feedback (like how large language models are redined by prompting people which which response they prefer). 

For robotics, such methods may include a human critic who compares a robot's trajectories and a reward model learns from these comparisons. An RL algorithm then is able to train the robot's policy based on predicted rewards from what is has learned.

This raises the following questions:
- **Alignment:** Do we learn the correct preference? A reward function only sees certain features, so it can be difficult to capture everything a human may care about.
- **Sample efficiency:** How much feedback is needed to learn preference? Every question interrupts the human, and humans may be inconsistent or dishonest in answering, so feedback needs to be used efficiently.

#### Robot Actions can Influence People
When robots and humans have to share the same space, the robot's actions will influence what humans do:
- **Robot actions change the state:** Humans have to adapt to what the robot is doing over repeated encounters.
- **Robot actions communicate intention:** For example, an autonomous car may slow down to signal to a human driver to pass.

#### Overview of Papers
**Both papers address one overall question:** How can robots understand what humans want and what humans do?

**What humans prefer: POLITE.** Human preferences are usually written as hand-crafted reward functions. However, human preference is complex and these relationships cannot be represented in reward functions of this nature. Standard preference learning only says which trajectory is better, but not why. POLITE allows humans to highlight positive and negative segments within a trajectory to indicate preference and uses these as regularization terms during reward learning.

**What humans intend to do: LILI.** Most robot learning focuses only on single-agent tasks, but real robots work alongside other agents and people who adapt their behavior to the robot in question over repeated interactions. This paper investigates how a robot can infer another agent's hidden strategy, and how to leverage this to influence other's behavior. LILI uses an encoder-decoder to learn a latent representation of the other agent's strategy, and then trains a strategy-conditioned RL policy that anticipated and influences other agents' future behavior.

---
# POLITE: Preferences Combined with Highlights in Reinforcement Learning
#### Simon Holk, Daniel Marta and Iolanda Leite <d-cite key="holk2024"></d-cite>
2024

## Motivation {#polite-motivation}
**Research question:** How can a robot learn a better reward function from human feedback when a preferred trajectory still contains behavior that the human dislikes?

Traditional RL depends on hand-crafted reward functions which struggle to capture complex human preferences, especially if there are several competing goals in one scenario. Preference-based RL allows for a human to compare two trajectories and says which one is preferred and why, which allows the robot to learn a reward function by balancing different goals involved in the environment and understanding the **causal relation**.

> Example: A robot navigating a corridor has two goals. (1) it must reach the star target and (2) respect the people around it. Trajectory X reaches the star but cuts in-between the people. Trajectory Y avoids interrupting the people but misses the star. The human perfers X > Y, yet they do not prefer every part of X.

{% include figure.liquid
   path="assets/img/2026-10-28-preference-influence/scenario.png"
   class="img-fluid rounded z-depth-1"
   caption="Figure 1: A social navigation example. Balancing reaching the end target against respecting the social needs of the surrounding people makes the reward hard to design by hand."
%}

**Problem with Prior Preference Learning.** A preferred trajectory does not mean that all of its actions are preferred. Standard preference learning raises the reward of the whole preferred trajectory, so the undesired parts (like cutting in-between the people) are never punished. As a result, the learned reward function has flaws.

**Intuition.** Two kinds of human feedback can be collected:
- The usual **trajectory-level preference** (X > Y)
- **Highlights** that point out which parts of the trajectory were desirable and which were not.

Where both will be used to update the reward function.

### Contribution
- Proposed highlights, which can be positive or negative, that let humans mark desired and undesired sections within a trajectory to help agent learn causal relation.

- Add highlights to preference learning as **regularization terms** on the reward model, which allows for finer-grained credit assignment.

- Demonstrate better sample efficiency in simulated MuJoCo tasks and a better-shaped reward in a user study with real human feedback.

## Related Works {#polite-related-works}
**Human-in-the-loop learning.** Prior studies let humans give evaluative feedback as scalar signals <d-cite key="knox2009"></d-cite>, natural language corrections <d-cite key="sharma2022"></d-cite>, or by segmenting a trajectory into desirable and undesirable parts <d-cite key="cui2018"></d-cite>. POLITE follows the same idea of allowing humans the chance to explain their choice, but builts it into preference learning so that the robot learns relevant information to avoid causal confusion with fewer queries.

**Learning from preference:** Preferenced-based RL <d-cite key="wirth2017"></d-cite> learns rewards through active query selection <d-cite key="sadigh2017"></d-cite>, from scratch <d-cite key="christiano2017"></d-cite>, bootstrapped with imitation learning <d-cite key="ibarz2018"></d-cite>, or from a pre-trained policy <d-cite key="lee2021"></d-cite>. In all of these prior studies, one label covers the entire segment. POLITE extends these works by adding highlights to show which parts of the segment are a part of the preferred label or not.

## Methodology {#polite-methodology}

### Problem Formulation
**Segments and Highlights.** A segment $\sigma$ is a section of a trajectory. A highlight $h$ is a short windown inside a segment that ends at the selected state $j$:

$$
h = \begin{cases} ((s_0,a_0),\ldots,(s_j,a_j)), & \text{if } j \le L \\
((s_{j-L},a_{j-L}),\ldots,(s_j,a_j)),  & \text{if } j > L \\
\varnothing, & \text{if no highlight}
\end{cases}
$$

where $L$ is the highlighted length, which is a hyperparameter. There are two kinds of highlights:
- **Positive highlight $h^+$:** behavior that needs to be rewarded.
- **Negative highlight $h^-$:** behavior that needs to be avoided.

Highlighting is also optional, so the human does not have to highlight any part of the segment in some cases.

**Highlighted Query.** Each request for human annotation is represented as a tuple:

$$hq = (\sigma_1, \sigma_2, \mu, h^+, h^-)$$

where $$\sigma_1, \sigma_2$$ are the two segments being compared and $$\mu$$ is the overall trajectory preference label (which segement is better than the other). The trajectory can have multiple segments, and segments can have multiple highlights. Each query contains at most one positive and one negative highlight *per segment*.

{% include figure.liquid
   path="assets/img/2026-10-28-preference-influence/representation_of_highlights.png"
   class="img-fluid rounded z-depth-1"
   caption="Figure 2: A segment with a positive highlight (green) and a negative highlight (red) of length L ending at state j."
%}

**Goal.** To learn a reward model $$\hat{r}_\psi(s,a)$$ that agrees with the humans preferences for the trajectory and their highlights, and then train a policy on that reward function.

### Framework

**Preference Loss (same as prior work).** The probability that the human prefers one trajectory over the other $$\sigma_1 \succ \sigma_2$$ is the exponential sum of the rewards of the preferred trajectory over the total predicted rewards for both of the trajectories:

$$
\hat{P}[\sigma_1 \succ \sigma_2] =
\frac{\exp\left(\sum_{\sigma_1} \hat{r}_\psi(s,a)\right)}
{\exp\left(\sum_{\sigma_1} \hat{r}_\psi(s,a)\right) + \exp\left(\sum_{\sigma_2} \hat{r}_\psi(s,a)\right)}
$$

The reward model is also trained with cross-entropy against the human labels, for the error between the predicted and actual human judgment:

$$
\mathcal{L}_{\text{pref}}(\hat{r}_\psi) =
-\sum_{(\sigma_1,\sigma_2,\mu)\in\mathcal{D}_q}
\mu(1)\log\hat{P}[\sigma_1 \succ \sigma_2] + \mu(2)\log\hat{P}[\sigma_2 \succ \sigma_1]
$$

**Positive Highlight Loss.** The positive highlight loss raises the predicted reward of the highlighted actions:

$$
\mathcal{L}_{+} = -\mathbb{E}_{h^+\sim\mathcal{D}_{hq}}
\left[\sum_{l=0}^{L}\lambda^{l}\,\hat{r}_\psi(s_{j-l},a_{j-l})\right]
$$

**Negative Highlight Loss.** The negative highlight loss lowers the predicted reward of the highlighted actions:

$$
\mathcal{L}_{-} = \mathbb{E}_{h^-\sim\mathcal{D}_{hq}}
\left[\sum_{l=0}^{L}\lambda^{l}\,\hat{r}_\psi(s_{j-l},a_{j-l})\right]
$$

The two losses here only differ in sign. The discounted $$\lambda^l$$ gives the most weight to the state at the highlighted moment $$j$$. States that are further back in the window will count less, which emphasizes the actions close to the highlighted moment.

**Total Loss.** The highlight loss terms act as regularizers for standard preference learning:

$$\mathcal{L}_{\text{POLITE}} = \mathcal{L}_{\text{pref}} + \alpha_{+}\mathcal{L}_{+} + \alpha_{-}\mathcal{L}_{-}$$

where $$\alpha_+$$ and $$\alpha_-$$ control how strongly each kind of highlight influences the reward.

**Training Process.** First, the policy is randomly initiallized and the initial reward model is trained. These steps are then repeated until convergence:
- **Step A:** Use the current reward model to update the policy with PPO, and collect segments $$\sigma$$.
- **Step B:** Show pairs $$(\sigma_1, \sigma_2)$$ to the human, who will give their preference $$\mu$$ and annotate highlights $$h^+$$ and $$h^-$$.
- **Step C:** Add the queries to the highlighted query dataset $$\mathcal{D}{hq}$$ and update the reward model with $$\mathcal{L}_{\text{POLITE}}$$.

{% include figure.liquid
   path="assets/img/2026-10-28-preference-influence/architecture.png"
   class="img-fluid rounded z-depth-1"
   caption="Figure 3: POLITE training loop. (A) policy rollouts, (B) human gives preference and highlight annotations, and (C) reward model updates."
%}

## Experiments {#polite-experiments}

### Experiment 1: Simulation (Synthetic Feedback)
**Hypotheses:**
- **H1:** POLITE is more sample efficient, so it needs fewer human queries than SOTA methods.
- **H2:** POLITE converges faster than other methods.

### Setting
**Task:** Walker2d and HalfCheetah, two standard MuJoCo environments in OpenAI Gym.

**Feedback:** A simulated human labels preferences with the true environment reward function (oracle). 10% of the labels are flipped to mimic a noisy human teacher. Highlights are generated by comparing to a threshold: segments with rewards above or below a set threshold are highlighted.

**Methods:** POLITE, PEBBLE <d-cite key="lee2021"></d-cite>, and a Preference Learning baseline <d-cite key="christiano2017"></d-cite> (POLITE without highlights: $$\mathcal{L}_{\text{POLITE}} = \mathcal{L}_{\text{pref}}$$).

<div class="row mt-3">
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid
       path="assets/img/2026-10-28-preference-influence/walker2d.gif"
       class="img-fluid rounded z-depth-1"
    %}
  </div>
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid
       path="assets/img/2026-10-28-preference-influence/halfcheetah.gif"
       class="img-fluid rounded z-depth-1"
    %}
  </div>
</div>
<div class="caption">
  The two MuJoCo environments, Walker2d (left) and HalfCheetah (right), used in the simulation experiments (source: Gymnasium documentation).
</div>

### Main Results
- Walker2d: POLITE with **200** queries beat the baseline with **800** queries and PEBBLE with **400** queries. With 400 queries, POLITE reaches nearly double the performance of the 800 query baseline.
- HalfCheetah: With an equal number of queries, POLITE outperforms PEBBLE and the baseline.

{% include figure.liquid
   path="assets/img/2026-10-28-preference-influence/sim_env_graphs.png"
   class="img-fluid rounded z-depth-1"
   caption="Figure 4: Reward curves on Walker2d (a) and HalfCheetah (b), and the ablation studies (c)."
%}

**Is this a fair comparison?** Not really. A highlighted query carries more information than a plain preference, because it also includes which part of the trajectory was favored or not. Comparing methods by number of queries therefore really favors POLITE, as it is getting richer information that the other methods cannot receive without more queries.

### Ablation Studies
**Highlight vs. Preferences.** The ablation study compares POLITE with *highlights only* and with *preference only*, each with 400 queries. Highlights only performs far worse than the preference only ablation, and combining both results in the best performance. **The preference is the main learning signal, and highlights work as regularizers that refine these signals.**

### Experiment 2: Real Human Feedback
- **H3:** POLITE learns a substantially different reward function and policy than baseline preference learning.

- **H4:** POLITE is better at identifying and rewarding sparse goal states because highlights provide additional shaping to its reward function.

### Setting
**Social navigation.** A simulated Pepper robot navigates through a corridor with three moving humans to reach a target. The study compares only POLITE and the baseline, using 400 preferences each. Both conditions start from the  same pool or segments and preference pairs, generated by the same policy, but POLITE also collects highlights. 40 participants were included in this study for annotating; 20 for POLITE and 20 for the baseline.

<div class="row mt-3">
  <div class="col-sm mt-3 mt-md-0">
    {% include video.liquid
       path="https://www.youtube.com/embed/Swk6xUsPMmM"
       class="img-fluid rounded z-depth-1"
       controls=true
    %}
  </div>
  <div class="col-sm mt-3 mt-md-0">
    {% include video.liquid
       path="https://www.youtube.com/embed/jFAapKvzJyI"
       class="img-fluid rounded z-depth-1"
       controls=true
    %}
  </div>
</div>
<div class="caption">
  The simulated Pepper robot navigates between walking people to get to goal.
</div>

### Results

- **Different reward functions (H3):** The baseline's learned reward is mostly flat. POLITE's reward increases as the robot gets closer to the goal (H4).

- **Diverging Policies:** The KL-divergence between the POLITE and baseline policies grows over training epochs, which shows that the two methods learn increasingly different behaviors.

- **Performance:** POLITE gets a higher environment reward than the baseline.

{% include figure.liquid
   path="assets/img/2026-10-28-preference-influence/reward_distribution_and_KL_graphs.png"
   class="img-fluid rounded z-depth-1"
   caption="Figure 5: Learned reward vs. corridor position (a), learned reward vs. distance to goal (b), KL-divergence over training epochs (c), and the environment reward (d)."
%}

## Conclusion {#polite-conclusion}

In this paper, preference-based RL is extended upon using positive and negative highlights. Humans then are able to give richer feedback by marking which parts of the trajectory they have a preference for, and these annotations are used within the reward function as regularization terms on top of the standard preference loss. With this finer-grained credit assignment, this method requires less queries in simulation. With real human feedback, the policy learns are more shaped reward that emphasizes the actions nearest to the goal. The addition of these highlights also makes human preferences more interpretable, as they show the causal features for driving behaviors.

## Limitations {#polite-limitations}
- **Limited highlight expressiveness:** Each query only allows one $$h^+$$ and $$h^-$$. A segment with serveral key events can only can only get credit for one desirable and one undesirable event.

- **Confounded human study:** The study split the participants into different groups of 20 people per condition. Reward differences may come from different people giving feedback rather than the highlight themselves.

- **Untested highlight length:** The highlight length $$L$$ is a hyperparameter, and the paper does not explore how it affects the performance nor how the baseline $$L$$ was decided upon.

---
# LILI: Learning Latent Representations to Influence Multi-Agent Interaction
#### Annie Xie, Dylan P. Losey, Ryan Tolsma, Chelsea Finn, and Dorsa Sadigh <d-cite key="xie2021"></d-cite>
2020

## Motivation {#lili-motivation}
**Research question:** How can a robot learn to influence another agent's behavior to achieve better coordination?

In multi-agent systems, agents continuously adapt to each other's behavior, making the environment non-stationary. Rather than merely reacting to others, an ego agent should anticipate their future strategies and learn how its own actions can influence them.

> Example: In air hockey, the ego agent can learn how its blocking behavior affects the opponent's future shooting directions and gradually guide the opponent toward shots that are easier to block.

{% include figure.liquid
   path="assets/img/2026-10-28-preference-influence/ego_fig.png"
   class="img-fluid rounded z-depth-1"
   caption="Figure 6: Air hockey example: The robot opponent has a hidden strategy (aiming for the right side of the table). The ego agent purposefully blocks in a way to influence the opponent to make shots that are easier to block."
%}
### Contribution
* Propose an RL framework that learns latent representations of other agents' strategies.
* Enable the ego agent to anticipate and influence future strategies through repeated interactions.
* Demonstrate improvements across simulated environments and real-world robotic air hockey.

## Related Works {#lili-related-works}
**Multi-Agent Reinforcement Learning.** Prior studies address non-stationarity through opponent modeling <d-cite key="raileanu2018"></d-cite>, role representations <d-cite key="losey2020"></d-cite>, centralized training <d-cite key="lowe2017"></d-cite>, and communication protocols <d-cite key="foerster2016"></d-cite>. LILI instead models other agents' strategies using compact latent representations.
**Influence Through Interactions.** Unlike methods that explicitly model opponents' policy updates <d-cite key="foerster2018"></d-cite> or introduce social influence rewards <d-cite key="jaques2019"></d-cite>, LILI learns influential behavior by maximizing long-term rewards without an additional influence reward.

## Methodology {#lili-methodology}

### Problem Formulation

**Strategy Affects Transitions and Rewards.** Let (z^i) denote the other agent's latent strategy during interaction (i). The environment is modeled as a hidden-parameter MDP: $$\mathcal{M}^i = \langle \mathcal{S},\mathcal{A},\mathcal{Z},\mathcal{T},R,H\rangle$$, where transitions $$\mathcal{T}(s'\mid s,a,z^i)$$ and rewards $$R(s,z^i)$$ depend on the latent strategy. Each interaction produces a trajectory: $$\tau^i={(s_1,a_1,r_1),\ldots,(s_H,a_H,r_H)}.$$ 

**Strategy Changes Between Interactions.** The other agent updates its strategy according to: $z^{i+1}\sim f(\cdot\mid z^i,\tau^i).$ Therefore, the ego agent's actions influence the interaction trajectory $\tau^i$, potentially affecting the other agent's future strategy.

**Goal.** Maximize cumulative discounted rewards across repeated interactions while accounting for strategy changes.

### Framework

{% include figure.liquid
   path="assets/img/2026-10-28-preference-influence/framework_fig.png"
   class="img-fluid rounded z-depth-1"
   caption="Figure 7: LILI framework: the encoder maps the previous interaction to a latent strategy (left). The encoder and decoder are trained on repeated interactions by predicting the next states and rewards (right)."
%}

In the figure above, on the left side, the ego agent leverages prior knowledge $\tau^{i-1}$ and uses an encoder to extract latent information, subsequently generating a policy based on that latent representation. To update this, the ego agent learns by sampling a pair of consecutive interactions and training the latent space to predict the latent information of the other agent.

Mathematically, the encoder model $$\mathcal{E}_\phi$$ models $$f(z^{i+1} \mid \tau^i)$$, an approximation of the latent dynamics $$f(z^{i+1} \mid z^i, \tau^i)$$. Since the latent strategy of the other agent determines how they will react to the ego agent's behavior, we decode the latent strategy $$z^{i+1}$$ using a decoder $$\mathcal{D}_\psi$$ that reconstructs the transitions and rewards observed during interaction $$i+1$$. Therefore, the objective function is to train both the encoder and decoder under the following maximum likelihood objective $$\mathcal{J}_{\text{rep}}$$:

$$
\max_{\phi,\psi}\sum_{i=2}^{N}\sum_{t=1}^{H}\log p_{\phi,\psi}
\left(s_{t+1}^{i}, r_t^{i}
\mid
s_t^{i}, a_t^{i}, \tau^{i-1}\right)
$$

where $$\mathcal{E}_{\phi}$$ produces a deterministic embedding $$z^i$$ given $$\tau^{i-1}$$, and $$\mathcal{D}_{\psi}$$ outputs a distribution over the next state $$s_{t+1}$$ and reward $$r_t$$ given the current state $$s_t$$, action $$a_t$$, and embedding $$z^i$$.

The encoder is particularly important here: we can use $\mathcal{E}_{\phi}$ to anticipate the other agent's next strategy. Besides learning a meaningful latent space, the next step is to jointly optimize with long-term rewards by finding the optimal policy.

**Influencing by Optimizing for Long-Term Rewards.** In an autonomous driving scenario, a human driver can adopt two strategies: $z_1$ (avoiding and passing the autonomous car) or $z_2$ (trusting and following it). The ego agent aims to encourage $z_2$, which is more beneficial for long-term coordination. To learn influential behavior, the ego agent optimizes its policy $\pi_\theta$ across repeated interactions:

$$
\max_{\theta} \sum_{i=1}^{\infty} \gamma^i \mathbb{E}_{\rho_{\pi_\theta}^{i}}
\left[\sum_{t=1}^{H} R(s,z^i)\right]
$$

where $\gamma\in[0,1)$ is a discount factorand $\rho_{\pi_\theta}^{i}(\tau)$ is the trajectory distribution induced by $\pi_\theta$ under dynamics $\mathcal{T}(s' \mid s,a,z^i)$.

Maximizing this reinforcement learning objective naturally motivates the ego agent to influence the other agent. Specifically, the ego agent learns to generate interactions $\tau^i$ that lead the other agent to adopt future latent strategies which the ego agent can exploit for higher rewards. The detailed algorithm for joint learning is described below.

{% include figure.liquid
   path="assets/img/2026-10-28-preference-influence/alg_1.png"
   class="img-fluid rounded z-depth-1"
   caption="Figure 8: The LILI algorithm: each interaction is run using a strategy inferred from the previous one and stored into a replay buffer. Representation and policy learning are then updated jointly from sampled interaction pairs."
%}

## Experiments {#lili-experiments}
### Setting
**Baseline:** SAC <d-cite key="haarnoja2018"></d-cite>, SLAC <d-cite key="lee2020"></d-cite>, LILAC <d-cite key="xie2020"></d-cite>, Oracle. 

**Method:** Two variants: LILI (No Influence) that train the latent network, while the ego agent is trained to make greedy decisions that maximize its intermediate expected return, while LILI enables the long-term, discounted sum of rewards across multiple interactions. **This comparison enable a better comparison for whether latent space is important or not, and how it affect towards different types of reward when targeting a policy.** 

**Task:** Point Mass, Lunar Lander, Driving (2D and CARLA). 
- Point map (sim), lunar lander (sim), driving (sim), air hockey (real)
- **Baselines:** soft actor critic, stochastic latent actor-critic, lifelong latent actor-critic, Oracle (knows the other agent's hidden intention -- optimal)

#### Point Mass Visualization
<div class="row mt-3">
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid
       path="assets/img/2026-10-28-preference-influence/pointmass_sac.gif"
       class="img-fluid rounded z-depth-1"
    %}
  </div>
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid
       path="assets/img/2026-10-28-preference-influence/pointmass_lilac.gif"
       class="img-fluid rounded z-depth-1"
    %}
  </div>
</div>
<div class="row mt-3">
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid
       path="assets/img/2026-10-28-preference-influence/pointmass_lili_no.gif"
       class="img-fluid rounded z-depth-1"
    %}
  </div>
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid
       path="assets/img/2026-10-28-preference-influence/pointmass_lili.gif"
       class="img-fluid rounded z-depth-1"
    %}
  </div>
</div>
<div class="caption">
  Point Mass. SAC (top left) vs. LILAC (top right) vs. LILI, no influence (bottom left) vs. LILI (bottom right).
</div>

#### Lunar Lander Visualization
<div class="row mt-3">
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid
       path="assets/img/2026-10-28-preference-influence/lunarlander_sac.gif"
       class="img-fluid rounded z-depth-1"
    %}
  </div>
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid
       path="assets/img/2026-10-28-preference-influence/lunarlander_lili_no.gif"
       class="img-fluid rounded z-depth-1"
    %}
  </div>
</div>
<div class="caption">
  Lunar Lander. SAC (left) vs. LILI, no influence (right)
</div>

#### Driving (CARLA) Visualization
<div class="row mt-3">
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid
       path="assets/img/2026-10-28-preference-influence/carla_sac.gif"
       class="img-fluid rounded z-depth-1"
    %}
  </div>
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid
       path="assets/img/2026-10-28-preference-influence/carla_lili_no.gif"
       class="img-fluid rounded z-depth-1"
    %}
  </div>
</div>
<div class="caption">
  Driving (CARLA). SAC (left) vs. LILI, no influence (right)
</div>


### Main Results
{% include figure.liquid
   path="assets/img/2026-10-28-preference-influence/simulated_envs.png"
   class="img-fluid rounded z-depth-1"
   caption="Figure 9: The simulated environments used for evaluation. Point Mass: reach a hidden target that moves outside of the circle. Lunar Lander: land at the hidden target that the other agent moves. Driving (2D): pass a car that is merging. Driving (CARLA): the same passing task where the ego agent controls only its steering angle."
%}

From the above figure, across different task, we see the importance of having a guided network for seemlessly interactions. One failure case is when the ego agent controls its steering angle, which can  lead to a crash between cars. This happens because the ego does not anticipate the other car's merge action, showing that interactions requires more than a single criteria.


### Ablation Studies
**Learnability.** As shown in below (left), the reward values has the positive slope across the increase number of interactions, which means the latent space is not contracdict to the policy, but comlement and improve over to make a more useful policy generated.

{% include figure.liquid
   path="assets/img/2026-10-28-preference-influence/exp_graphs.png"
   class="img-fluid rounded z-depth-1"
   caption="Figure 10: Point Mass results: The reward over interaxctions (left), where LILI is the closest to the Oracle. Ego trajectories between LILAC, LILI (no influence), and LILI (right)."
%}

**Dynamic Strategy.** From the three figures on Point Mass visualizations above, we see that the ego agent of LILAC could not go into the border of the circle to catch the other agent, but stay inside, while LILI's two variants can achieve that faster. 
**Intermediate vs. Discounted Reward.** As shown in the below figure, we see that for the air hockey scenario, across 200 interactions, the ego agent receives a bonus reward for blocking left, therefore it tends to prioritize more on left and middle compared to the right side. At convergence, LILI (no influence) gets an average reward of $1.0 \pm 0.05$ per interaction, where LILI get $1.15 \pm 0.05$.

{% include figure.liquid
   path="assets/img/2026-10-28-preference-influence/hockey_graphs.png"
   class="img-fluid rounded z-depth-1"
   caption="Figure 11: Real-robot air hockey results: blocking success rate over interactions (left), distribution of where the opponent shoots (middle and right)."
%}

## Conclusion {#lili-conclusion}
In this paper, a framework for multi-agent interaction that represents the low-level policies of non-stationary agents with high-level latent strategies is proposed, and incorporates these strategies into an RL algorithm. 
They show that robots were able to anticipate how their behavior would affect another agent’s latent strategy, and actively influenced that agent for more seamless co-adaptation.

## Limitations {#lili-limitations}
* **Strategy Assumptions:** The framework assumes latent strategies remain fixed within each interaction and exhibit predictable dynamics across interactions.
* **Generalization:** Rapid or unpredictable changes in other agents' strategies may reduce the reliability of learned representations.
* **Human Interaction:** The influence-aware policy was brittle against human opponents because it relied on precise paddle-edge blocking. More robust human-in-the-loop evaluation is needed.

## Overall Conclusion
Together, these papers show how a robot can learn hidden preferences and intentions behind human behavior.

**Human preference.** A trajectory-level preference can be used to tell the robot which behavior humans prefer, but not what about the action is truly valued. Highlights were used to fill this gap by identifying specific segments that should be rewarded or avoided. This shapes a reward function that better aligns with complex human preferences, needs fewer queries, and makes preferences more interpretable.

**Human interaction.** Other agents in the world are not static; they adapt to everything within their environment, including the robot in question after every interaction. By learning a latent representation of the agents' hidden strategies, the robot is able to predict how those strategies will change and how to react in the future. There is also some insight on the difference between a current reward and future reward. LILI, with no influence, maximizes the immediate reward within each interaction, while LILI maximizes the discounted reward across multiple interactions. This way it learns to shape the other agent's future behavior for its benefit.

**Connecting the two.** In both papers, the robot is able to learn something it cannot directly observe. POLITE learns human preference from explicit feedback, and LILI learns the other agent's strategy from implicit feedback, which is by how the agent behaves. This shows that real human-robot interaction involves both a human that shapes the robot's behavior through feedback, and a robot that shapes other agents through its actions; which is known as co-adaption. To deploy robots into real-world scenarios, they will need to be capable of doing both for effective, safe, and interpretable interaction with humans and other agents.

---

## Discussion Questions and In-Class Clarifications
This section summarizes questions raised during the presentation and supplements the recorded responses with clarifications from the papers and the reporter team’s analysis.

### 1. Questions about POLITE

#### 1.1. What distinguishes a segment from a highlight, and how do the associated hyperparameters affect learning?

This question is about the distinction between segments and highlights, including why the method introduces segments rather than simply using highlights. The presenters described trajectories as being divided into segments, with each segment containing a positive and a negative highlight.

The paper’s formulation is more specific: a **segment** is a portion of a trajectory used in a preference comparison, while a **highlight** is a subsequence within the preferred segment. Each query compares two segments and allows at most one positive and one negative highlight in the preferred segment. Either or both highlights can be omitted. Thus, saying that every segment contains two highlights is not totally accurate, and the formulation does not require an exhaustive partition of a trajectory into segments.

Segments provide the behavioral context that users compare, while highlights identify particular events within that context. They are therefore different levels of feedback rather than interchangeable units.

The class also asked about the hyperparameters governing segmentation and highlighting. Segment length controls the amount of context shown in a comparison, while highlight length controls the temporal scope of local feedback. Short windows may omit relevant context, whereas long windows may include unrelated actions. The number of queried segment pairs also affects feedback coverage and annotation effort. These are distinct choices, and the recorded exchange did not establish how their trade-offs were evaluated.

#### 1.2. Does reducing the number of queries reduce human annotation effort?

A key question raised whether comparing methods by query count adequately captures the cost of human feedback. A POLITE query can contain both a preference and positive or negative highlights, whereas a baseline query contains only a preference. The queries therefore differ in both information content and annotation effort.

The paper reports that completing 20 queries and the tutorial took an average of 24.83 minutes with highlights, compared with 18.93 minutes for the baseline. However, it does not directly measure the total annotation time required to reach comparable policy performance.

Comparing query counts is informative, but it does not fully account for the additional supervision and effort associated with highlights. The reported reduction in queries should therefore not be interpreted as an equivalent reduction in human effort. Comparing policy performance under a fixed annotation time budget would provide a more direct evaluation of this trade-off.

#### 1.3. Why do highlights alone perform much worse than highlights combined with preferences?

Addressing the concern, it was noted that the paper reports this ablation result without providing a detailed explanation. At a high level, they described preferences as the main learning signal and highlights as additional regularization terms that improve reward learning.

Specifically, the two feedback types impose different constraints on the reward model. Preferences require the cumulative predicted reward of the preferred segment to exceed that of the alternative. Highlights encourage higher rewards for selected positive subsequences and lower rewards for selected negative subsequences, but do not directly specify which complete segment is better overall.

With highlights alone, the model receives local feedback on selected subsequences but lacks the explicit segment-level comparisons provided by preferences. It may therefore learn to reward or penalize highlighted events without adequately ranking complete behaviors. Combining the two provides both an overall comparison and more localized feedback about desirable and undesirable behavior.

This complementary-information explanation is the reporter team’s interpretation, rather than a mechanism directly established by the experiments. The weaker performance of highlights alone is the observation to be explained, it does not by itself establish why the resulting policies fail.

### 2. Questions about Learning and Influencing Latent Intent (LILI)

#### 2.1. How should the latent strategy be interpreted, and how does it relate to HMMs and VAEs?

HMMs and VAEs provide useful analogies for understanding LILI's latent representations, but neither fully describes the framework.

Like an HMM, LILI uses hidden variables to explain observable behavior. However, latent strategy transitions depend on interaction trajectories, including the ego agent's actions.

Unlike a standard VAE, LILI uses a **deterministic encoder** and a **predictive objective without KL regularization**. The learned embedding is not necessarily an interpretable intention; it primarily supports prediction and decision-making.

#### 2.2. What allows LILI to influence future strategies rather than merely react?

The key distinction is the **optimization horizon**.

**LILI (No Influence)** maximizes rewards within the current interaction, whereas **LILI** maximizes discounted rewards across interactions. Consequently, LILI can sacrifice immediate rewards to influence the other agent toward more beneficial future strategies.

In **Point Mass**, LILI deliberately controls its ending position to influence the target's next location, rather than only pursuing the current target.

However, influence may provide little additional benefit when different future strategies yield similar rewards, as observed in Lunar Lander and driving tasks.

#### 2.3. Why predict the next interaction, and how does prediction reliability affect performance?

LILI predicts transitions and rewards in the next interaction rather than reconstructing the previous trajectory. This encourages the latent representation to capture information about how the other agent's strategy evolves.

The policy receives the **inferred embedding**, while the decoder provides a **prediction-based training signal**.

However, performance depends on the **accuracy and generalization** of the latent representation. Unpredictable opponents may cause incorrect strategy inference and suboptimal actions.

The human air hockey experiment highlights a related robustness challenge: the influence-aware policy did not transfer reliably to human opponents, although the experiment does not establish whether latent prediction errors caused this failure.
