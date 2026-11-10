---
layout: distill
title: "Social Interaction, Cooperation"
description: "Multi-agent coordination, game-theoretic formulations of human-robot teaming, and social conventions in shared workspaces."
date: 2026-11-02
future: true
htmlwidgets: true
ready: false

# Reporter team authors
authors:
  - name: "Pei-Chi Chen"
  - name: "Yuezhang Chen"
  - name: "Junyan Huang"
  - name: "Yi Ping"
  - name: "Yiqun Yin"

bibliography: 2026-11-02-social-interaction-cooperation.bib

toc:
  - name: "Introduction"
  - name: "Social interactions as recursive MDPs"
    subsections:
          - name: "Motivation"
          - name: "Background"
          - name: "Method"
          - name: "Experiments"
          - name: "Conclusion"
  - name: "HABIT: Human-Aware Behavior and Interaction Training Dataset for Robot Manipulation"
    subsections:
          - name: "Motivation"
          - name: "Key ideas"
          - name: "Background"
          - name: "Task workflow"
          - name: "Principles"
          - name: "Setup"
          - name: "Results"
          - name: "Limitations"
  - name: Discussion Questions

---

## Introduction

> **Topic Overview**: Multi-agent coordination, game-theoretic formulations of human-robot teaming, and social conventions in shared workspaces.

This lecture report covers the **Social Interaction, Cooperation** session in *Learning for Interactive Robots (CS 6501, Fall 2026)* at the University of Virginia.

Multiple robots work together to achieve higher goals. But building a multi robot/agent system does not simply mean putting multiple robots or agent together. It often requires 3 Cs: **Coordination， Cooperation, Collaboration**.

**Structures of multi-human/robot interaction:**
{% include figure.liquid
   path="assets/img/2026-11-02-social-interaction-cooperation/Structures-of-multi-humanRobot-interaction.png"
   class="img-fluid rounded z-depth-1"
   alt=""
   caption=""
   zoomable=true
   avoid_scaling=true
%}

**Roles of interaction**:
- supervisor
- Operator
- Teammate
- Mechanic/Programmer
- Bystander
- Mentor
- Information Consumer
- The roles are quickly evolving: assistant shared autonomy, etc

**Why do robots need social skills:**
- Robots are leaving the lab and entering shared spaces: Robots will work in an environment with humans
- People are not obstacles: Robot needs to interact with human
- Intent is hidden: Robot has to infer why human acts
- Handover and shared workspace: Robot may replace or collaborate with human

**This leads to today's problem in two angles:**
- How should a robot reason about other agents' goals?
- Where does human-aware behavior come from

## Social interactions as recursive MDPs
### Motivation

Social interaction requires reasoning recursively about how another agent's goals instead of only focusing on the goal itself. For example, both robots want to water the same tree themselves. Sharing this physical destination does not imply cooperation: one robot may also try to prevent the other from completing its task. A robot that only infers where its partner is going can therefore miss why that partner acts. Recognizing an intention to hinder changes how it should respond.

Motivated by the need to equip robots with social skills and the ability to infer other agents’ social goals, this paper introduces **Social MDPs** to give such interactions an explicit planning formulation. An agent combines a **physical goal**, concerning what it wants to accomplish in the environment, with a **social goal**, concerning another agent's reward. Helping and hindering become objectives that can be balanced against the agent's own task and action costs. Because the other agent may also reason socially, this formulation requires nested estimates of goals and rewards.

### Background

**Markov Decision Process (MDP).** An MDP models sequential decisions when the agent observes the current state. It can be written as

$$
\mathcal{M}=\langle S,A,T,R\rangle,
$$

where $$S$$ and $$A$$ are state and action spaces, $$T(s'\mid s,a)$$ gives transition probabilities, and $$R(s,a)$$ specifies reward. A policy $$\pi(a\mid s)$$ selects actions to maximize expected cumulative reward. In the standard MDP, the reward represents the physical goal of the agent.

**Partially Observable MDP (POMDP).** When the state is not directly observable, the model adds observations:

$$
\mathrm{POMDP}_i
=\langle S,A_i,T_i,\Omega_i,O_i,R_i\rangle.
$$

Here, $$\Omega_i$$ is the observation space, and $$O_i(o_i\mid s',a_i)$$ gives the probability of an observation after action $$a_i$$ leads to state $$s'$$. 

**Interactive POMDP (I-POMDP).** I-POMDPs is the extensions of POMDPs that recursively model other agents' beliefs about the state of the world. An agent reasons about what another agent believes, including that agent's beliefs about other agents. This supports predicting actions based on the other agent's perspective and available information.

The two-agent formulation is:

$$
\mathrm{I\text{-}POMDP}_i
=\langle IS_i,A,T_i,\Omega_i,O_i,R_i\rangle,
\qquad IS_i=S\times M_j.
$$

Here, $$IS_i$$ is the **interactive state space**, combining the physical state $$S$$ with possible models $$M_j$$ of agent $$j$$. The action space $$A=A_i\times A_j$$ contains joint actions. Agent $$i$$ maintains a belief over interactive states; when its model of $$j$$ includes $$j$$'s beliefs about agent $$i$$, the belief structure becomes recursive. <d-cite key="gmytrasiewicz2005framework"></d-cite>

**Connection to Social MDPs.** I-POMDPs recursively model agents' beliefs about the world state, whereas Social MDPs recursively model **their estimated reward functions**. Social MDPs use this reward dependence to express helping and hindering while keeping physical states fully observable.

### Method

**1. Social MDP** For agent $$i$$ interacting with agents in $$J$$, a Social MDP describes its physical task and social objectives, with a maximum reasoning level $$l$$. The model is defined as:

$$
M_i^l=\langle S,A,T,\chi_{iJ},g_i,R_i^l,\gamma\rangle.
$$

- $$S$$ is the environment's state space, with $$s\in S$$ denoting a particular state.
- $$A=A_J$$ is the joint action space of the agents in $$J$$. Each agent chooses an individual action $$a_i\in A_i$$, and $$a_J$$ collects the actions taken by all agents.
- $$T(s'\mid s,a_J)$$ gives the probability of reaching state $$s'$$ from state $$s$$ after the joint action $$a_J$$.
- $$\chi_{iJ}$$ collects agent $$i$$'s social goals toward the other agents. Each $$\chi_{ij}$$ describes its social objective with respect to agent $$j\ne i$$, such as helping or hindering that agent.
- $$g_i$$ is agent $$i$$'s physical goal.
- $$R_i^l$$ is agent $$i$$'s reward function at reasoning level $$l$$. It incorporates the agent's estimates of other agents' rewards.
- $$\gamma\in(0,1)$$ is the discount factor controlling the weight of future rewards.

**2. Recursive reward.** The immediate reward of an agent $$i$$ at level $$l$$ is computed as follows:

$$
\begin{aligned}
R_i^l(s,a_J,\chi_{iJ},g_i)
&=r_i(s,a_i,g_i)+\sum_{\substack{j\in J\\j\ne i}}
\chi_{ij}\!\left(
\widetilde R_j^{i,l-1}
\left(s,a_J,\widetilde\chi_{jJ}^{i,l},\widetilde g_j^{i,l}\right)
\right)-c(a_i).
\end{aligned}
$$

- $$r_i(s,a_i,g_i)$$ is the physical reward associated with agent $$i$$'s own goal $$g_i$$.
- $$c(a_i)$$ is the cost when agent $$i$$ takes action $$a_i$$.
- $$\widetilde R_j^{i,l-1}$$ is agent $$i$$'s estimate of agent $$j$$'s reward, treating $$j$$ as a level $$l-1$$ agent.
- $$\widetilde\chi_{jJ}^{i,l}$$ represents agent $$i$$'s estimates, at reasoning level $$l$$, of agent $$j$$'s social goals toward other agents.
- $$\widetilde g_j^{i,l}$$ is agent $$i$$'s estimate, at reasoning level $$l$$, of agent $$j$$'s physical goal.
- $$\chi_{ij}$$ transforms agent $$j$$'s estimated reward into a contribution to agent $$i$$'s reward. The framework permits an arbitrary transformation.

This paper uses a linear transformation, written here as $$\chi_{ij}(x)=w_{ij}x$$:

- $$w_{ij}>0$$ favors **helping**, as increasing agent $$j$$'s estimated reward benefits agent $$i$$.
- $$w_{ij}<0$$ favors **hindering**, as decreasing agent $$j$$'s estimated reward benefits agent $$i$$.
- $$w_{ij}=0$$ removes the social reward contribution associated with agent $$j$$.

The magnitude of $$w_{ij}$$ controls the importance of the social objective relative to the physical reward. A stronger social weight can justify additional action costs, while a weaker weight may leave the agent prioritizing its own physical task.

**3. Planning for Social MDPs.** Planning combines the agent's immediate reward with the expected consequences of future interactions. The state-action value is defined as:

$$
\begin{aligned}
Q_i^l(s,a_J,\chi_{iJ},g_i)=R(s,a_i,\chi_{iJ},g_i)+\gamma\sum_{s'\in S}
T(s,a_J,s')V_i^l(s',\chi_{iJ},g_i).
\end{aligned}
$$

Here, $$R$$ denotes the immediate reward defined above. The transition notation $$T(s,a_J,s')$$ denotes the same probability as $$T(s'\mid s,a_J)$$. To evaluate future states, agent $$i$$ must account for uncertainty about the partner's goals and actions. The expectation of Q-value is:

$$
\begin{aligned}
V_i^l(s',\chi_{iJ},g_i)
&=\max_{a_i'\in A_i}
\mathbb{E}_{\widetilde g_j^{i,l},\widetilde\chi_{jJ}^{i,l},a_j'}
\left[Q_i^l(s',a_J',\chi_{iJ},g_i)\right].
\end{aligned}
$$

The expectation uses the inferred probabilities of the partner's physical and social goals, together with its estimated policy.

**3.1 Updating social and physical goals of other agents.** At time $$t=0$$, the model uses uniform distributions over candidate physical and social goals. At subsequent steps, it updates these estimates using observed behavior and the partner's estimated policy from the previous time step.

**Social-goal update.** The belief about the partner's social goal is updated as:

$$
\begin{aligned}
&P\!\left(\widetilde\chi_{ji}^{i,l,t}
\mid s^{t-1},a_J^{t-1}\right)\propto
P\!\left(\widetilde\chi_{ji}^{i,t-1}
\mid s^{t-2},a_J^{t-2}\right)\times
\sum_{\widetilde g_j^{i,l,t-1}}
P\!\left(a_j^{t-1}\mid s^{t-1},
\widetilde\chi_{ji}^{i,l,t-1},\widetilde g_j^{i,l,t-1}\right)\times T(s^{t-1},a_J^{t-1},s^t).
\end{aligned}
$$

- $$\widetilde\chi_{ji}^{i,l,t}$$ is agent $$i$$'s estimate, at level $$l$$ and time $$t$$, of agent $$j$$'s social goal toward $$i$$.
- The previous belief is reweighted by the likelihood of the observed action $$a_j^{t-1}$$, summing over candidate physical goals $$\widetilde g_j^{i,l,t-1}$$.
- $$T(s^{t-1},a_J^{t-1},s^t)$$ accounts for the transition produced by the previous joint action.

**Physical-goal update.** The physical goal is inferred from the observed state history while marginalizing over the unknown social goal:

$$
\begin{aligned}
&P\!\left(\widetilde g_j^{i,l,t}\mid s^{1:t-1}\right)\propto
\int_{\widetilde\chi_{ji}^{i,l,t}}
P\!\left(s^{1:t-1}\mid
\widetilde g_j^{i,l,t},\widetilde\chi_{ji}^{i,l,t}\right)\cdot
P\!\left(\widetilde g_j^{i,l,t}\right)
P\!\left(\widetilde\chi_{ji}^{i,l,t}\right)
\,d\widetilde\chi_{ji}^{i,l,t}.
\end{aligned}
$$

- $$\widetilde g_j^{i,l,t}$$ is agent $$i$$'s estimate of agent $$j$$'s physical goal, and $$s^{1:t-1}$$ is the observed state history.

These updates allow behavior to be interpreted in terms of both physical and social objectives, rather than attributing every action to a physical destination alone.

**3.2 Estimating social policies of other agents.** Updated goal estimates are used to predict the other agent's actions. Agent $$i$$ estimates agent $$j$$'s policy using the lower-level Q-function:

$$
\begin{aligned}
&\widetilde\psi_j^{i,l-1}
\left(s,a_J,\widetilde\chi_{jJ}^{i,l},\widetilde g_j^{i,l}\right)=\mathrm{Softmax}\!\left(
Q_j^{l-1}\left(s,a_J,\widetilde\chi_{jJ}^{i,l},\widetilde g_j^{i,l}\right)
\right).
\end{aligned}
$$

The softmax uses a temperature parameter $$\tau$$ to control how strongly predicted actions favor higher values. Evaluating agent $$j$$ at level $$l-1$$ requires modeling agent $$i$$ at level $$l-2$$, continuing until a level 0 MDP is reached.

**3.3 Recursive planning procedure.** The nesting levels determine what the agent can reason about:

| Reasoning level | Model of the other agent | Social reasoning enabled |
|---|---|---|
| Level 0 | No nested social model; solve a regular MDP. | Pursue a physical goal. |
| Level 1 | Treat the other agent as a level 0 agent. | Help or hinder the other agent's physical goal. |
| Level 2 | Treat the other agent as a level 1 agent. | Account for the other agent's social intentions, such as an attempt to help or hinder oneself. |

{% include figure.liquid
   path="assets/img/2026-11-02-social-interaction-cooperation/social-mdp-figure-1-recursive-reasoning.png"
   class="img-fluid rounded z-depth-1"
   alt="A yellow robot at reasoning level 2 models a red robot at level 1, which in turn models the yellow robot at level 0. Arrows connect the nested policy computations."
   caption="Figure 1: An example of recursively solving Social MDP for the
yellow robot at level 2 in a two-agent interaction scenario."
   zoomable=true
   avoid_scaling=true
%}
At level 0, the agent solves a regular MDP. At higher levels, it updates the other agent's goal estimates, recursively computes lower-level policies for candidate goals, and uses those estimates to calculate its own reward and action values. It then selects its own action by maximizing expected value over the other agent's possible goals and actions. This process repeats at each time step as new behavior is observed.

### Experiments

**1. Setup** The experiments use a virtual 2-agent grid world <d-cite key="tejwani2022social"></d-cite>. There are six things in each scene: a red robot, a yellow robot, a red water bucket, a yellow water bucket, a tree, and a flower. A robot waters a plant by pushing a water bucket to it. The yellow robot pushes the yellow bucket twice as fast as the red bucket, and the red robot is the other way around. The physical goal is to water the plant by yourself, not just to have the plant watered.

**2. Scenarios** The paper tests 98 scenarios. They cover:

- the same or different physical goals for the two robots,
- hinder, help, or neutral social goals,
- a physical/social scaling factor of $$-2, -1, -0.5, 0, 0.5, 1,$$ or $$2$$.

Negative values mean hindering and positive values mean helping. In class, the presenters explained a few of these values:

- $$-2$$: very unhelpful.
- $$0$$: the robot does not care about the other robot's physical goal at all.
- $$0.5$$: it cares a little.
- $$1$$: it weighs the physical and the social goal about the same.
- $$2$$: very helpful.

Each scenario is run with level 1 and level 2 reasoning, mainly from the point of view of the yellow robot. At level 1, the yellow robot assumes the red robot has a physical goal, but does not know its social goal. At level 2, the yellow robot also tries to guess the red robot's social goal.

**3. Example** In the example from the slides, the red robot has the flower as its physical goal and a social goal of 2. The yellow robot has the tree as its physical goal and a social goal of 0. With level 1 reasoning, the yellow robot is a bit confused about what the red robot is trying to do. With level 2 reasoning, the yellow robot infers that the red robot is trying to help. It meets the red robot partway, so it does not have to take as many steps, and it finishes faster.

In the motivation part, the presenters also showed a video where both robots want to water the tree and both want to hinder the other. When the yellow robot cannot model the red robot's social goal, it gets very close to watering the tree with the red bucket. Then it sees the red robot pushing the yellow bucket, and it does not understand that the red robot is pushing away the bucket that is easier for it to push. It chases after the red robot, and the red robot comes back and waters the tree first. When the yellow robot can model the red robot's social goal, it just pushes the red bucket right to the tree from the beginning.

**4. Human evaluation** Twelve human subjects watched the videos of all scenarios at both levels, $$98 \times 2 = 196$$ videos. They rated what they saw as the robots' physical and social goals. These ratings were compared with the estimates of the yellow robot in Social MDP, and with two baselines: inverse planning and a cue-based model. The cue-based model is basically a supervised learning approach. Because this is a virtual environment, the ground truth goals are also known.

**5. Results**

- Social MDP matches humans well. Its correlation with the human ratings is high, while the two baselines do pretty poorly.
- Looking at each time step, at the beginning nobody is really sure what is happening. As more steps are seen, the estimates lean toward one answer. In the example shown, the physical goal is probably the flower, and the social goal moves toward hindering the yellow robot.
- For the social goal, the human scores and Social MDP are much closer to the ground truth than inverse planning and the cue-based model.
- For the physical goal alone, the presenters noted that inverse planning seems to do better.

### Conclusion

**Overall findings**

- Social MDP does zero-shot social recognition and planning. In comparison, the cue-based model needs supervision.
- It has a strong alignment with the human mental model, as shown by the human study.
- As the instructor said at the start of the lecture, the idea of this paper is that a mathematical formulation lets social concepts like helping or hindering transfer to new situations, without relearning the underlying mechanism.

**Limitations**

- The environment is a simple virtual environment and is fully observable. The paper does not use the partially observable version.
- All goals are predefined, which is not what we would see in the real world, where people's goals may change.
- There is a limited number of agents and environment objects, so it is not as noisy as the real world.
- At each step, the model has to enumerate all possible physical and social goals, recursively. The presenters pointed out that this becomes very complicated, and likely intractable, in a real-world scenario. Approximations would be needed, for example sampling possible goals or using a neural network.
- It was not tested with humans.

Even with these limits, the presenters saw Social MDP as a good baseline. The next paper gets into studies with humans.

## Discussion Questions:

* Social MDPs use a linear weight X on the other agent’s reward (negative means hinder, positive means help). Do you think there are real social behaviors can’t be captured by a linear weighting?

   Linear weighting does not fully capture all of social behaviors. Dynamic linear weight may be a solution to improve robustness. Conditional and context-dependent social goal “Help unless it costs me more than X” , for example, can’t be represented with a fixed weight. But, altering the reward function could be a solution.

   We also discussed putting nonlinear operation in reward function may help. But it will result in the complexity of computing your reward function.

* One way to build social robots is to put structure in explicitly (model-driven). Another is to
   learn behavior from large amounts of interaction data, as with learned policies trained on
   demonstrations (data-driven). Which approach is likely to generalize better to new people
   and situations?

   Model-driven approaches work well on specific situation but fail to generalize the variances of the same task. Combining method may create a possibility for better generalizations.
   Each generalizes along a different axis, and a hybrid approach is likely the best

* HABIT’s data protocol treats acting before the human is ready as a failure (precondition
violation), and operators are told to act only in reaction to their partner. However, some
robot policies anticipate human actions and act early, thus reducing waiting time. What
separates good anticipation from a violation? Does reactive-only data cap how fluent a
robot can be?

   Good anticipation means preparing reversible actions while delaying irreversible ones until the
   human's intent is clear. Reactive-only will encourage grounded behavior, but may limit fluency. 
   
   Sometimes you may not wanna robot to anticipate. Because robots may fail to anticipate and infer human behavior. It is also very risky for robot to act before human since it does not act optimally most of the time.

* Training a VLA on human-present demonstrations improves results but doesn’t solve
collaboration. Collaborator tasks stay low and precondition violations persist. What is
missing from today’s VLAs for working with people? Are VLAs the right architecture for
collaborative tasks?
   
   VLAs provide a strong perception-to-action foundation, but often lack explicit representations of a
   human partner, task state, and temporal preconditions.

--

### Report Status

This report is currently being prepared and will be updated by the student reporter team following the lecture session and classroom discussion. Please check back soon!
