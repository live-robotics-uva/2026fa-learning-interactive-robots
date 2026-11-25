---
layout: distill
title: "Safety, Trust"
description: "Human trust dynamics, trust-aware planning, temporal logic constraints, and verifiable safety constraints in robotics."
date: 2026-11-23
future: true
htmlwidgets: true
ready: true

# Reporter team authors
authors:
  - name: "Huu Binh Ta"
  - name: "Duc Anh Nguyen"
  - name: "Tristan Scott"

bibliography: 2026-11-23-safety-trust.bib

toc:
  - name: "Introduction"
    subsections:
      - name: "Trust"
      - name: "Safety"
      - name: "Safety vs. Trust"
  - name: "Planning with Trust for Human-Robot Collaboration"
    subsections:
      - name: "Trust Motivation"
      - name: "Trust Related Works"
      - name: "Trust Methodology"
      - name: "Trust Experiments"
      - name: "Trust Conclusion"
      - name: "Trust Limitations"
  - name: "Plug in the Safety Chip: Enforcing Constraints for LLM-driven Robot Agents"
    subsections:
      - name: "Plug-in Motivation"
      - name: "Plug-in Related Works"
      - name: "Plug-in Methodology"
      - name: "Plug-in Experiments"
      - name: "Plug-in Conclusion"
      - name: "Plug-in Limitations"
  - name: "Overall Conclusion"
  - name: "Discussion Questions and In-Class Clarifications"
    subsections:
      - name: "Questions about “Planning with Trust”"
      - name: "Questions about “Plug in the Safety Chip”"
      - name: "Discussion questions"
  - name: "References"

---
*All authors are affiliated with the University of Virginia.*

## Introduction

This lecture report covers the **Safety, Trust** session in *Learning for Interactive Robots (CS 6501, Fall 2026)* at the University of Virginia.

> **Topic Overview**: Control barrier functions, reachability analysis, human trust dynamics, and verifiable safe interaction in robotics.

### Why does safety and trust matter in HRI?

We have previously focused on how robots can improve their performance within social settings using many different methods, but being intelligent or capable does not always mean being safe or trustworthy.

As robots move into homes and shared workplaces, their mistakes and misunderstandings will affect the people around them. For a robot to be capable, it can only useful if it behaves safely and people working with it are willing to rely on it a healthy amount. For real deployment, robots must *interact directly* with humans, and the decisions they make can result in *serious consequences* that could potentially harm people or the environment around them. Successful collaboration therefore requires that the robot perform *safe actions* and an *appropriate level of trust* is maintained within the interaction.

Although, this problem has a lot of social and situational nuance, and is not as simple as it may seem. To fully understand the layers to this topic, we will first introduce and define trust and safety within HRI.

### Trust

Trust can be defined as a human's expectation or confidence of how the robot will behave in the future. This expectation is mainly shaped by these three factors:
- **Perceived ability:** Can the robot do it?
- **Past experiences:** Has it succeeded before? Have my previous experiences with the robot been good?
- **Perceived risk:** What if it fails? Will there be harm to me?

The true robot's capability may not align with how the human understands it or how willing he or she is to rely on the robot. Trust in part has to do with perception and is not objective or common between any two people. The three factors listed are a common basis, but trust is actually much more complicated than that.

#### What topics are relevant for trust in robotics?

Trust is a hidden goal, and there is not much theory about how it works. So far, trust has been based in connections to other topics that are fostered in robotics <d-cite key="kok2020trust"></d-cite>, but there is still no clear metric for how to measure trust or much less understand all the possible factors that go into devloping and maintaining trust.

{% include figure.liquid
   path="assets/img/2026-11-23-safety-trust/trust_factors.jpg"
   class="img-fluid rounded z-depth-1"
   caption="Figure 1: A network showing the catagories involved in trust research within robotics."
%}

One major question is how does one gauge if there is too much or too little trust? Both overtrust and undertrust can have major consequences and can defeat the purpose of having an automated system.

#### Trust calibration: More trust is not always better

The goal for trust in HRI is that human trust should match what the robot is  actually capable of, meaning that there should be an *appropriate, calibrated* amount of trust. The trust should not be maximized.

- **Undertrust:** The human does not trust the system enough and rejects capable assistance, which reduces team efficiency and can even put the situation at risk.
> Example 1: The Honda Point Diaster (1923). During a weather event, a Navy squadron's new piece of technology, radio direction-finding (RDF), indicated they were off course. The pilot ignored this warning, which caused a ship wreck.
\\
> Example 2: Crash of USAir 1016 (1994). During a severe thunderstorm, the airport's ground-based windshear alert system issued three warnings to air traffic control. Because the system often gave false alarms, the controller did not pass on the warnings to the pilots. The plane then flew into the windshear and crashed, resulting in the dealths of 37 people.

- **Overtrust:** The human ignores the limitations of the robot, which can increase safety risk. In high-stress (life-threatening) situations, people may trust robots too much because they assume the robot has information they do not, even if the robot may not perform the right action. People also show this kind of behavior with LLMs and AI chatbots, often trusting the information without checking it.
> Example: Georgia Tech emergency evacuation study (2016). During a simulated fire, participants followed an emergency guide robot instead of marked exits, even when they had witnessed the robot performing poorly earlier in the study <d-cite key="robinette2016overtrust"></d-cite>.

#### Robot Trustworthiness vs. Human Trust.
The two ideas about trust that are commonly mixed up are the following:
- **Robot Trustworthiness.** A property of the robot that allows it to prove itself worthy of trust (based on its performance, safety, robustness, privacy, transparency).

- **Human trust.** A congnitive property of the users/stakeholders. It emerges from situations between the human and robot that involve uncertainty, caution, or vulnerability. Trust is also multifaceted and related to many sources of the robot's capability, and is also dependent on a history of observed events.

A trustworthy robot does not automatically earn trust, and a trusted robot is not necessarily automatically deserving of human trust.

### Safety

Safety in HRI can be split into two catagories:
- **Physical Safety:** aviod collisions, prevent human injuries, maintain safe distances.

- **Task-Level Safety:** respect operational rules, avoid prohibited actions, follow safety constraints. This will be the focus for this report.

Historically, one of the earliest approaches to this problem was to keep robots as far from humans as possible while operating (such as in a cage). While this comes from earlier robotics, it is still sometimes used today in factory settings.

#### Logic Constraints for Rule Enforcement

To enforce rules about how a task should be completed, robots need a language to follow that can describe order. Temporal logic describes a sequence of actions/states that the robot should follow when working to complete a task.

- **Propsitional logic:** specifies the goal as a single state.
> Example: touch the red block.

- **Temporal logic:** specifies the goal as a sequence over states, or a pattern that states must follow. This can be used to constrain the behavior of the robot.
> Example: touch the blocks inside the room and then go out.

**Operators for temporal logic.** Directions in temporal logic can be represented using operators in a language that the robot can understand.
- **X (next):** "Open the door and then get milk" ➔ $$\text{OpenDoor} \land X \, \text{GetMilk}$$
- **G (globally true, always):** "Always stop when you see a red light" ➔ $$G(\text{RedLight} \rightarrow X \, \text{Stop})$$

**Planner with safety monitoring.** Once the rules are written formally, the agent has to consider an aditional safety monitor along with the original planner to check if its decisions are safe. This may cause the robot to replan a motion that fits to these constraints, or abort if it cannot find a safe path.

### Safety vs. Trust

**Trust (human property).** "Will the human rely on the robot?" 
- Based on human perception and drives the human's choice to either rely on or intervene with the robot. 

**Safety (system property).** "Can the robot operate safely?" 
- Evaluates the robot's actual behavoior and physical limits.

Ultimately, their relationship is a "what came first?" situation and results in social feedback loop. There is a balance between trust and safety that robots must be sensitive to if they are to be deployed in collaboration settings.

{% include figure.liquid
   path="assets/img/2026-11-23-safety-trust/interaction_loop_fig.png"
   class="img-fluid rounded z-depth-1"
   caption="Figure 2: A diagram showing the human-robot interaction loop and how trust and safety constantly are changing the relationships within the collaborative team."
%}


#### Dual Challenge for AI Deployment
- **Goal 1: Maximizing performance (trust-based).** Robots must be effective collaborators, which requires modeling and managing hidden human states like trust. This will be the focus of the first paper.

- **Goal 2: Guaranteeing non-negotiable safety (constriant-based).** Robots must never violate critical safety constraints ("the don'ts"), which requires verifiable, formal methods. This will be the focus for the second paper.

---
# Planning with Trust for Human-Robot Collaboration
#### Min Chen, Stefanos Nikolaidis, Harold Soh, David Hsu, and Siddhartha Srinivasa <d-cite key="chen2018"></d-cite>
2018

## Motivation {#trust-motivation}
**Research question:** Can a robot reason about human trust when choosing its actions to improve long-term collaboration?

Trust is essential for HRI collaboration because it affects how much humans will rely on or intervene with the robot. In a collaborative task, the human has the decision at any point in time to let the robot act or to take over. This decision depends heavily on how much trust the person has for the robot, so a mismatch between the human's perception and the robot's true ability hurts overall collaborative task performance for the team.
> Example: A human and robot work together to clear objects off a table. Even if the robot is completely capable of moving all the objects, participants with low initial trust in the robot will often stop it from moving the wine glass if it goes for it first, considering it too risky.

{% include figure.liquid
   path="assets/img/2026-11-23-safety-trust/clear_table_fig.png"
   class="img-fluid rounded z-depth-1"
   caption="Figure 3: A human and robot work together to clear a table. The human intervenes to stop the robot from picking up the wine glass due to low trust."
%}

**Capability is not enough.** In the example above, the robot's ability to complete the task is not the problem. Rather, the human's lack of faith in the robot is. Every time that the human takes over the task when the robot could have done it, the work is shifting back onto him or her, and reduces the benefit of having a robot assistant.
    
**Issues with prior trust models.** Previous approaches have focused on only estimating trust by inferring how much a person trusts the robot from their interaction history. However, an estimate alone does not tell the robot how to respond to improve overall team performance.

**Why is trust hard to plan with?**
- **It cannot be observed.** The robot never directly observes trust, only indirect indications of it, so any estimate carries uncertainty.

- **Behavior is ambiguous.** When a human intervenes, it may not be the case that it was done out of low trust in general or just caution about that fragile object in particular. One decision alone does not reveal trust with complete certainty.

- **It's not constant.** Every success or failure, or action in-between, can influence the human's trust, so a model that describes a person well at the beginning may start to be wrong a few steps later.

- **It depends on the history so far.** The human's next decision could depend on the entire interaction history, which continues to grow and may be too long of a record for the robot to plan efficiently.

**Goal.** The robot should choose actions that make the team perform the best long-term. Building trust is worthwhile when it leads to the human relying on the robot an appropriate amount, but maximizing trust is not the goal itself.

### Contributions

- **The Trust-POMDP framework**, which models human trust as a latent variable inside of Partially Observable Markov Decision Process (POMDP) so the robot can (i) infer the human's trust through interaction, (ii) reason about how its actions affect human behavior, and (iii) chose actions that maximize long-term team performance.

- **A trust dynamics model**, which predicts the evolution of human trust over time as a result of the robot's actions, and **human decision model**, which predicts how trust connects to human actions, both learned from data.

## Related Works {#trust-related-works}

**Trust in social settings and automation.** In social science, trust is commonly considered to be composed from ability, integrity, and benevolence <d-cite key="mayer1995"></d-cite>, but trust in automation differs because automation lacks intentionality in the ways that trust between people does <d-cite key="lee2004"></d-cite>. Like prior works, this paper makes the assumption that the human does not expect the human to deceive them, so that trust depends on the robot's perceived task ability.

**Measuring and modeling trust.** Trust has previously been measured with binary <d-cite key="hall1996"></d-cite>, continuous <d-cite key="desai2012,lee1992,xu2016"></d-cite>, and ordinal scales <d-cite key="hoffman2013"></d-cite> such as Muir's questionnaire <d-cite key="muir1990"></d-cite>, as well as real-time measures like the Area Under Trust Curve <d-cite key="desai2012"></d-cite>. Other works model the temporal dynamics of trust, how trust changes over time, using time series analysis <d-cite key="lee1992"></d-cite>, case-based reasoning <d-cite key="floyd2015"></d-cite>, online probabilistic trust inference <d-cite key="xu2015"></d-cite>, or generating explanations to improve trust <d-cite key="wang2016"></d-cite>. This paper instead uses a trust model to choose actions to maximize task performance, not trust.

**POMDPs for human-robot collaboration.** POMDPs <d-cite key="kaelbling1998"></d-cite> provide a framework for reasoning over a human's hidden mental state, such as in autonomous driving scenarios among pedestrians <d-cite key="bai2015"></d-cite> or planning that uses this information to leverage human actions <d-cite key="sadigh2016"></d-cite>. Nikolaidis et al. infer the human type or preference online <d-cite key="nikolaidis2015"></d-cite> and model human-robot mutual adaption <d-cite key="nikolaidis2017"></d-cite>. This paper extends these ideas by linking the whole interaction history to the human's policy through trust dynamics.


## Methodology {#trust-methodology}

### Problem Formulation
**Human-robot team model.** The human-robot team is modeled as a Markov Decision Process (MDP) with world state $$x \in X$$, robot action $$a^R \in A^R$$, and human action $$a^H \in A^H$$. The human observes the robot's current action and then chooses his or her own action. Given a robot policy $$\pi^R$$ and human policy $$\pi^H$$, the value (the expected total discounted reward) is:

$$
v(x_0 \mid \pi^R, \pi^H) = \mathbb{E}_{a^R_t \sim \pi^R,\, a^H_t \sim \pi^H}
\left[\sum_{t=0}^{\infty} \gamma^t\, r(x_t, a^R_t, a^H_t)\right].
$$

However, the robot does not know the human's policy in advance, and so it computes the optimal policy in expectation over human policies:

$$
\pi^{R*} = \arg\max_{\pi^R} \mathbb{E}_{\pi^H}\, v(x_0 \mid \pi^R, \pi^H).
$$

**Complication with history.** For this formulation, the human's policy $$\pi^H(a^H_t \mid x_t, a^R_t, h_t)$$ is dependent on the entire history of the human-robot interaction:

$$
h_t = \set{x_0, a^R_0, a^H_0, x_1, r_1, \dots, x_{t-1}, a^R_{t-1}, a^H_{t-1}, x_t, r_t},
$$

which can grow arbitrarily long and makes the optimization increasingly more difficult.

**Insight: trust as a summary of history.** Trust is a compact approximation of the interaction history. Assuming that trust can be represented as a single scaler random variable $$\theta$$, the human policy becomes:

$$
\pi^H(a^H_t \mid x_t, a^R_t, \theta_t) = \pi^H(a^H_t \mid x_t, a^R_t, h_t).
$$

### Task Setup: Table Clearing
To understand the rest of the framework, it seems best to define the main table-clearing task so that the reward function $$R$$ is understandable. This will also help explain how the models were trained later in the upcoming section.

The testing task has a human and a robot clearing *five items* off a table: three water bottles, one open can of fish, and a wine glass. The items are chosen so that the robot's possible mistake gets progressively worse, from being harmless (dropping a closed plastic water bottle) to expensive (a broken wine glass). For each of the robot's actions, it reaches for an item, and the human has the option to decide if they will *let the robot pick up the object* or *intervene*. This is repeated until nothing is left on the table.

{% include figure.liquid
   path="assets/img/2026-11-23-safety-trust/reward_function_table.png"
   class="img-fluid rounded z-depth-1"
   caption="Figure 4: The reward function for the table-clearing task."
%}

Where SP-success indicates the robot successfully clearing the object from the table, SP-fail is when it fails, and IT is when the human intervenes. The rewards make it so that the robot's success is worth more than a human intervention, since it saves the human the effort. Letting the robot pick up the bottle is risk-free, while letting it pick up the wine glass is the most risky.

### Framework

The trust-POMDP is built from two learned components: a model that predicts how the human's trust changes after each observation (**trust dynamics model**), and a model that predicts whether the human will intervene given their trust (**human decision model**).

#### Trust Dynamics Model
After each time the robot takes an action, the human's trust updates based on the performance $$e_{t+1}$$, which corresponds to if the robot succeeded at picking up an object or if the human intervened. The method in the paper models this evolution as a linear Gaussian system:

$$
P(\theta_{t+1} \mid \theta_t, e_{t+1}) = \mathcal{N}\left(\alpha_{e_{t+1}}\theta_t + \beta_{e_{t+1}},\; \sigma_{e_{t+1}}\right),
$$

where $$\alpha_{e_{t+1}}$$ and $$\beta_{e_{t+1}}$$ are learned linear coefficients given what the performance outcome is. Using a normal distribution instead of a single value lets the model account for uncertainty in trust that may differ person to person.

{% include figure.liquid
   path="assets/img/2026-11-23-safety-trust/Muir_questionnaire_table.png"
   class="img-fluid rounded z-depth-1"
   caption="Figure 5: The four questions asked on a 1-7 Likert scale to participants."
%}

**Trust metric.** Since trust is not directly observable, the ground-truth labels come from the following Muir's questionnaire, which asks the human participants four questions to produce a score on a 1-7 scale. This questionnaire was based before and after each robot action. The questionnaire score $$\theta^M_t$$ is then treated as a noisy observation of what the true trust is:

$$
\theta^M_t \sim \mathcal{N}(\theta_t, \sigma^2).
$$

#### Human Decision Model 
There are two types of human behavior models: the trust-free human behavior model and trust-based human behavior model. Both versions assume the human follows a softmax rule, where the human makes a choice to either stay put or intervene probabilistically with probability $$P_t$$. Intervening will give a reward of 0, but staying put is more risky, with the expected reward being:

$$
b_j r^S_j + (1-b_j) r^F_j
$$

where $$r^S_j$$ and $$r^F_j$$ are the rewards when the robot either succeeds or fails. With just the two options, the softmax rule simplifies to the sigmoid $$S(x) = \frac{1}{1+e^{-x}}$$. The two versions of the behavior model only differ in where the human's belief $$b_j$$ comes from.

**Trust-free model.** The human's belief $$b_j$$ that the robot will succeed on the object $$j$$ is fixed and never updates:

$$
P_t = S\left(b_j r^S_j + (1-b_j) r^F_j\right), \qquad 
a^H_t \sim \mathcal{B}(P_t).
$$

**Trust-based model.** The human's belief that the robot will succeed depends on the current trust they have in the robot $$\theta_t$$, that come from the trust dynamics model:

$$
b^t_j = S(\gamma_j \theta_t + \eta_j), \qquad
P_t = S\left(b^t_j r^S_j + (1-b^t_j) r^F_j\right), \qquad 
a^H_t \sim \mathcal{B}(P_t).
$$

where $$\gamma_j$$ and $$\eta_j$$ are learned coefficients for each object, and $$\mathcal{B}$$ is a Bernoulli distribution.


#### Training
The data collected came from 76 Amazon Mechanical Turk workers. Each person watched a video of the robot attempting to pick an object up, filled out the questionnaire before and after every robot action and chose whether to intervene or not. The unknown parameters, $$\gamma_j, \eta_j$$ and $$\sigma$$,in the trust dynamics model were learned with Bayesian inference through Monte Carlo sampling.

{% include figure.liquid
   path="assets/img/2026-11-23-safety-trust/trust_transition_fig.png"
   class="img-fluid rounded z-depth-1"
   caption="Figure 6: Trust transition matrices representing the change in trust given observations of the robot's performance. The trust tends to stay constant or decrease slightly when the human intervenes (top), and increases when the human witnesses successful actions (bottom)."
%}

#### Trust-POMDP
The two models are combined into a POMDP with augmented state $$s = (x, \theta)$$, made up of the fully-observed world state $$x$$ and the partially observed human trust $$\theta$$. The robot maintains a belief $$b$$ over the human's trust, and its policy maps the belief and world state to an action $$a^R = \pi^R(b_t, x_t)$$. The optimal policy is computed with the SARSOP algorithm <d-cite key="kurniawati2008"></d-cite>.

The robot now balances two objectives through its actions:
- **Exploitation:** maximize the team's immediate reward.

- **Exploration:** reveal and change the human's trust so that future actions are rewarded better.

{% include figure.liquid
   path="assets/img/2026-11-23-safety-trust/graphical_model_fig.png"
   class="img-fluid rounded z-depth-1"
   caption="Figure 7: The graphical representation of trust-POMDP (left) and the team interaction flowchart (right)."
%}

#### Myopic Baseline
This is the short-sighted policy used for comparison within the experiments, and it plans with the trust-free human behavior model. Since it holds the belief that each person's probability of intervening never changes, it gains nothing from earning trust initially. Instead, it grabs for the most valuable items on the table first (glass, then can of fish, then bottles). 

{% include figure.liquid
   path="assets/img/2026-11-23-safety-trust/sample_runs_fig.png"
   class="img-fluid rounded z-depth-1"
   caption="Figure 8: The myopic policy's action will be intervened when it tries to pick up the glass in front of a cautious person because it has not built up trust first."
%}

## Experiments {#trust-experiments}
During the experiments, the robot actually never fails to pick up an object from the table, but participants are not told this. In doing the experiments in this way, the results are from how much the human trusts the robot to do its task, and not the true performance of the robot.
### Experiment 1: Online Video Experiment
**Hypothesis:**
- **H1:** The performance of teams in the trust-POMDP condition will be better than of the teams in the myopic condition.

### Setting
**Task:** Participants watched a clip of the robot reaching for an item, and then decide whether to step in or not. They then watch what happens after their decision, and then rate their trust using the Muir's questionnaire. The robot did not use this score, it tracked trust purely through its dynamics model.

**Participants:** 201 participants from Amazon Mechanical Turk: 101 for Trust-POMDP and 100 for myopic.

**Methods:** Trust-POMDP and the myopic baseline.

### Experiment 2: Real Table-Clearing Task
**Hypothesis:**
- **H2:** The performance of teams in the trust-POMDP condition will perform better than teams in the myopic condition.

### Setting
**Task:** The same table-clearing task, procedure, and questionnaire from the first experiment, but the participants interacted with a real robot in-person.

**Participants:** 20 participants: 10 for Trust-POMDP and 10 for myopic.

**Methods:** Trust-POMDP and the myopic baseline.

### Main Results

**Team performance.** Across both experiments, teams that were paired with trust-POMDP earned significantly more reward than the teams paired with the myopic robot, which confirms H1 and H2.

{% include figure.liquid
   path="assets/img/2026-11-23-safety-trust/results_fig.png"
   class="img-fluid rounded z-depth-1"
   caption="Figure 9: Intervention rate per object (left), mean trust over time (middle), and intervention rate vs. trust (right) for the online (top) and real-robot (bottom) experiments."
%}

**Intervention rate (left of graph).** For the online experiment, interventions for the can fell by 54% and 31% on the glass. With the real robot, the effect was much larger: with no interventions for the can (100% drop) and interventions for the glass dropped to 71%. The trust-POMDP was able to get the human to trust it with higher risk objects by clearing the low-stake items from the table first, before going for the can and glass.

**Trust evolution (middle of graph).** With each successful action from the robot, both policies saw that trust rose from the interaction. What is surprising is that the trust-POMDP did not end with a trust that was significantly higher than the myopic robot, even if the human involved intervened less. The authors point to the "primacy effect" as the reason for this, where early successes in the interaction history shaped trust more than later ones.

**Trust influences intervention (right of graph).** Low trust tended to lead to high intervention rates, which is especially visible for the online experiment. This means that the real participants behaved the way that the trust-based model predicted, where more trust meant there would be fewer interventions. This rate also depended on the object itself and the reward associated to it, as people let the robot take the bottle even if the trust was low, but only let it take the glass when trust was high enough.

## Conclusion {#trust-conclusion}

This paper incorporates human trust into robot decision-making by treating it as a latent state that summarizes the entire interaction history between the human and robot. With learned models of how trust evolves during interaction and how it then drives human actions, the robot can both estimate and deliberately shape the human's trust for it.

**Key takeaways:**
1. Teams paired with trust-POMDP outperforms teams paired with the myopic robot.
2. By accounting for trust, the robot can choose actions that optimize the long-term performance of the team.
3. Trust is used as a means to improve task performance, it is not a score that it should try to maximize.


## Limitations {#trust-limitations}
- **No robot failures:** Because the robot succeeded every time, the experiments never show how the model handles any drop in trust after the human sees the robot make a real mistake.
- **One value for trust:** Trust is only represented using a single scalar value which oversimplifies it. A richer model may fit real people better.
- **Robot movement is ignored:** How the robot moves and goes to perform an action can also shape trust (like a legible or deceptive motion), but this model has no way to capture this.
- **Hand-designed rewards:** The reward function is manually specified (seen in the reward table), and doing this for more complex tasks may become too difficult.
- **Static capabilities:** The model assumes that the robot's abilities never change, which may not be true in real deployment cases.
- **Small and simple study:** The real-robot experiment had 20 participants involved in a five-object task, so it is not clear how  well the trust model generalizes to other tasks or robots.



---
# Plug in the Safety Chip: Enforcing Constraints for LLM-driven Robot Agents 
#### Ziyi Yang, Shreyas S. Raman, Ankit Shah,and Stefanie Tellex  <d-cite key="yang2023"></d-cite>
2023


## Motivation {#plug-in-motivation}

**Research question:** How can robots reliably obey safety constraints?

* Large Language Models (LLMs) emerge as the promising way for robotics and planning tasks by leveraging the world knowledge and reasoning abilities obtained from pretraining. 

* Teaching "dos" and "don'ts" are not much attention though pretty important to avoid prohibited actions and ensuring the compliance of performing tasks.

* This opens the question: How can we enable more safety LLM usage for robot actions? under a collaborative environment?

> Example: In the demonstration below, given some safety constraints, the method in this paper will translate NL to LTL formulas, monitoring the agent’s decision-making process, and reprompting with reason of violation for re-planning.

{% include figure.liquid
   path="assets/img/2026-11-23-safety-trust/demonstration_fig.png"
   class="img-fluid rounded z-depth-1"
   caption="Figure 10: Demonstration of how the safety chip works in practice, translating NL to LTL formulas, monitoring the agent's decision-making process according to rules, and reprompting with the reason for why a rule has been violated."
%}

## Contributions

* Proposed a safety constraint module for customizable constraints and integrated the proposed module into an existing LLM agent framework.

* A fully prompting-based approach that supports predicate syntax for translating Natural Language (NL) to Linear Temporal Logic (LTL) and explaining violation of LTL specification in NL.

* A formal method-based action pruning and feedback for active re-planning for LLM agents.

* Deployed the whole system in an embodied environment and on real robot platforms and conducted baseline comparisons.


## Related Works {#plug-in-related-works}

**LTL for Safety in Robotics.** There are three main research directions: (i) **Runtime verification and monitoring** <d-cite key="foughali2020runtime, barringer2004rule"></d-cite> use temporal logic to check whether system executions satisfy predefined constraints, (ii) **Specification elicitation and human-robot interaction** <d-cite key="kressgazit2021formalizing"></d-cite> formalize requirements to support safe robot behavior, and (iii) **Constraint monitoring and enforcement** <d-cite key="yang2023, mao2021plc"></d-cite> apply temporal logic to identify violations and enforce safety requirements. This paper integrates LTL-based monitoring with natural-language constraints and LLM-based replanning.

**LLM Agents.** Recent works <d-cite key="ahn2022saycan, huang2022inner, huang2022zeroshot"></d-cite> demonstrate the potential of LLMs for robotic planning and decision-making. Huang et al. <d-cite key="huang2023grounded"></d-cite> introduce learned scoring functions for grounded decoding, while Liang et al. <d-cite key="liang2023code"></d-cite> generate Python programs for robot control. Vemprala et al. <d-cite key="vemprala2024chatgpt"></d-cite> explore code-based robot control with human feedback, and Xie et al. <d-cite key="xie2023translating"></d-cite> translate natural-language instructions into formal planning goals using PDDL. This paper introduces an external safety module that can be integrated into existing LLM agents without retraining.

**Translation between NL and LTL.** Previous approaches range from sequence-to-sequence models <d-cite key="gopalan2018seq2seq"></d-cite> to LLM-based translation <d-cite key="chen2023nl2tl, pan2023dataefficient"></d-cite>. However, limited training data and generalization to out-of-distribution formulas remain challenging. Cherukuri et al. <d-cite key="cherukuri2022explainable"></d-cite> investigate LTL-to-NL translation to improve the interpretability of formal specifications. This paper uses prompting-based NL-to-LTL translation and generates natural-language explanations of constraint violations to support replanning.

## Methodology {#plug-in-methodology}
### Problem Formulation

For an existing LLM agent making decisions based on in-context knowledge $$l_{\text{agent}}$$, we enforce a series of constraints in natural language $$\{c_i\}$$ concerning safety aspects by encoding into LTL formulas $$\{\varphi_i\}$$ and composing them into one single formula $$\varphi = \bigwedge_{i=0}^{\infty} \varphi_i$$, stored as automaton $$A$$.

Responding to safety constraint query $$Q$$, the module generates a response $$W$$ via a language model $$P(W \mid l_{\text{env}}, l_A)$$, where $$l_{\text{env}}$$ is the domain knowledge, $$l_A$$ is an explanation of constraint violation generated from $$P(l_A \mid l_{\text{agent}}, Q, T_{0:i})$$. To monitor LLM agent executing a high-level task $$T$$, splitting into subtasks or actions $$\{t_i\}$$, reject unsafe actions based on the partial trajectory $$\{t_0, t_1, \ldots, t_i\}$$ with $$A$$, and regenerate $$t_i'$$ after modifying $$P(t_i' \mid l_{\text{agent}}, T_{0:i-1})$$ to $$P(t_i' \mid l_{\text{agent}}, T_{0:i-1}, l_A)$$.


### Architecture

The framework consists of three modules:
* **Translator:** Converts natural-language constraints into LTL formulas.
* **Constraint Monitor:** Uses an automaton to validate proposed actions and reject unsafe ones.
* **Query System:** Generates natural-language explanations of violations to help the LLM replan.

{% include figure.liquid
   path="assets/img/2026-11-23-safety-trust/system_diagram_fig.png"
   class="img-fluid rounded z-depth-1"
   caption="Figure 11: The system overview showing the complete architecture including the constraints from NL, automaton, decision-making process of the LLM agent, and reprompting queries."
%}

### Identifying Safety Constraints 

**Stage 1.** We asked the human designer to specify the constraints using natural language. We automatically map the language to LTL formula. 

**Stage 2.** Human experts verify the generated LTL formulas to ensure they correctly represent the intended safety requirements. This supports the reliability needed for safety-critical systems, although ISO 61508 compliance is outside the scope of this paper.

> NL to LTL translation example. Referring expressions are extracted and grounded into predefined propositions, and lifted NL utterances are translated into a lifted formula, then combined to produce the final result.

{% include figure.liquid
   path="assets/img/2026-11-23-safety-trust/NL_to_LTL_fig.png"
   class="img-fluid rounded z-depth-1"
   caption="Figure 12: Natural-Language-to-LTL translation, using the temporal logic operators from the introduction."
%}

### Enforcing Safety Constraints

**Action validation:** The automaton $A$ checks the partial trajectory $T_{0:i}$ after each proposed action $t_i$. An action is accepted only if all state transitions are valid and the resulting automaton state has a path to an accepting state. If the agent terminates, the final state must already be accepting.

**Action pruning:** If a proposed action violates the constraints, it is rejected, and the LLM is asked to regenerate an action based on the previous valid trajectory $T_{0:i-1}$. This is action-level pruning rather than token-level masking.

**Motivation for reprompting:** Simply rejecting unsafe actions does not change the LLM's generation distribution. Therefore, the framework provides natural-language feedback to guide the LLM toward safer actions.

### Reprompting with Large Language Models

Assuming we have perfect truth value functions and transition model, any violation of the safety constraints can be detected in advance, and we can mask out the unsafe action and ask the LLM agent to re-generate the plan from the last step. 

However, it does not shift the distribution towards the ground truth, or does not provide information to the LLM agent, motivating provide information through updating context, and derive the process called reprompting. 

> An example can be found in the following figure on the left, where text in grey is the prefix in the prompt, black is generated by the LLM agent, and green is error messages generated for reprompting. task pecification and examples are removed for simplicity.

### Queryable Safety Constraints

The query system serves the purpose of assisting LLM agents in effective reprompting. Motivating by prior works <d-cite key="raman2022corrective, yao2023react"></d-cite>, this approach prompts the query system to reason over explanations for validation results and feeds the output to the LLM agent, introducing an inductive bias from the query system. Similar to the translation system, the query system relies entirely on prompting.

Agent query is only sent to the query system when the LLM agent violates given safety constraints in its next action. After a violation is detected by the constraint monitor, safety constraints in NL, valid actions and invalid actions, and their corresponding truth value changes are provided in the prompt together with task specifications, which explicitly instruct the language model to explain the reason for the violation. 

Importantly, the query system does not determine whether an action is safe. The automaton performs validation through trajectory progression, while the LLM generates natural-language explanations to support replanning.

> An example prompt for generating the first error message in the prompt can be shown in the following figure on the right.

<div class="row mt-3">
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid
       path="assets/img/2026-11-23-safety-trust/reprompting_img.png"
       class="img-fluid rounded z-depth-1"
    %}
  </div>
  <div class="col-sm mt-3 mt-md-0">
    {% include figure.liquid
       path="assets/img/2026-11-23-safety-trust/ex_prompt_img.png"
       class="img-fluid rounded z-depth-1"
    %}
  </div>
</div>
<div class="caption">
  Figure 13: (Left) an example of reprompting. (Right) an example prompt for generating the first error message.
</div>

## Experiments {#plug-in-experiments}

### Experiment 1: VirtualHome
### Tasks
1. **VirtualHome:** simulated household robot tasks
2. **Four Room:** rules about room visits and ordering
> Example: visit bathroom before kitchen.
3. **Mobile Manipulation:** room + object-state rules
> Example: if fridge is opened, close it later.
4. **20 household tasks across 2 environments;** 1–5 rules per task

### Methods
1. **Base Model:** GPT-4 plans without safety rules
2. **NL Constraints:** rules included in the GPT-4 prompt (LLM must reason about and obey rules itself)
3. **Safety Chip:** same planner + external LTL monitor (Block unsafe actions; explain violation; replan)
4. **Expert verification:** clarified NL rules or checked LTL formulas

### Experiment 2: Physical Robot
**Task:** The authors deploy Safety Chip on a Boston Dynamics Spot robot in an indoor environment containing 23 objects and landmarks.
* Two mobile manipulation tasks with up to 10 safety constraints per task.
* Constraints cover six avoidance and trigger patterns.
* Each task is executed twice with LLM temperature set to zero.

**Methods:** Code as Policies, NL Constraints, and Safety Chip.
* All methods receive predefined manipulation skills and access to object states and the robot planner.

The experiments assume expert-verified LTL constraints and no mechanical failures.
### Main Results

#### VirtualHome Success vs. Safety

We denote success is executable goal reached and safety means that no rule volations.

From the Table, we see that Safety Chip achieve the success rate and safety across two benchmark, though it does not achieve 100% success but it outperforms the others in safety. Therefore, the key takeaway is high task success does not imply safety, and balancing this is important.


{% include figure.liquid
   path="assets/img/2026-11-23-safety-trust/success_and_safety_table.png"
   class="img-fluid rounded z-depth-1"
   caption="Figure 14: Table showing the average success rate and safety rate of the VirtualHome experiments (Four Room and Mobile Manipulation)."
%}


#### Four Room in VirtualHome and Mobile Manipulation

Next, consider the average safety rate with respect to the increase of constraints in VirtualHome. As from the figure we see that Safety Chip achieves 100% safety at every constraint count, while the others fail sharply with the increase of number of constraints. For example, at 5 constraints: base model 0%; NL-only 20–40%*

**Takeaway: formal monitoring scales better than prompting**


{% include figure.liquid
   path="assets/img/2026-11-23-safety-trust/av_safety_rate_fig.png"
   class="img-fluid rounded z-depth-1"
   caption="Figure 15: The average safety rate vs. the number of constraints for the VirtualHome experiments (Four Room and Mobile Manipulation)."
%}

#### Physical Robot

Safety Chip achieves 100% safety and 98% task success, compared with 34% safety for NL Constraints and 55% for Code as Policies.

When the number of constraints exceeds five, the safety rates of NL Constraints and Code as Policies drop to 0% and 40%, respectively.

In three additional tests involving tasks that conflict with safety constraints, Safety Chip correctly aborts execution without entering unsafe states, while the baselines still attempt to complete the tasks.


{% include figure.liquid
   path="assets/img/2026-11-23-safety-trust/physical_robot_table.png"
   class="img-fluid rounded z-depth-1"
   caption="Figure 16: Table showing the average success rate and safety rate of the physical robot experiment."
%}

## Conclusion {#plug-in-conclusion}

This paper introduces Safety Chip, a plug-in framework that combines natural-language constraints, LTL-based runtime monitoring, and LLM-based reprompting to improve the safety of robot agents.

**Key takeaways:**
1. LLMs struggle to consistently follow safety constraints, especially as their complexity increases.
2. Formal monitoring provides more reliable safety enforcement than prompting alone.
3. Separating safety verification from LLM reasoning enables safer planning without retraining the underlying agent.

**Future Work:** Interactive constraint specification, improved natural-language explanations from automata, and neural-network-based truth-value functions for open-vocabulary propositions.

## Limitations {#plug-in-limitations}
- **Strong assumptions:** Safety guarantees rely on accurate truth-value functions and state-transition models.
- **Human verification:** Correct NL-to-LTL translation still requires expert verification, limiting scalability.
- **Limited expressivity:** Safety constraints depend on predefined propositions and LTL representations.
- **Evaluation scope:** Experiments focus on structured household environments and a limited set of constraint patterns, rather than unrestricted real-world scenarios.


## Overall Conclusion
Together, these two papers show the complementary components of what it takes for a robot to work in collaborative settings alongside people: earning the right amount of trust from humans, and never breaking the rules that keep them safe.

**Human trust.** A robot's true capability means very little if the human working with it cannot trust it enough to let it act. By treating trust as  a hidden state, the trust-POMDP lets the robot estimate how much the human trusts it and predicts when they will went to step in. The robot then can use this understanding to plan using this social information. Rather than going right to the highest-reward action, the robot can first prove itself worthy of trust using low-risk objectives and save the risky ones until the human is ready to put some faith into it. This menas that to improve long-term performance, the goal for robotics is a calibrated level of trust, rather than a maximum one.

**Safety constraints.**  An LLM agent can complete as task while still violating rules along the way, and simply adding these rules to its prompt stops being so effective as the number of constraints increases or the constraints become more complex. The safety chip seperates planning from the rule enforcement, where the NL rules are translated into LTL, an automaton checks every proposed action before it runs, and then violations are explaine back to the LLM so it can replan its action. This makes it so that the method is not dependent on the LLM to reason correctly about the rules itself.

**Connecting the two.** Both of the papers add structure around a robot's decision-making to handle something that the robot itself cannot simply learn to come to a solution. The trust-POMDP models a hidden human state to make the robot better at collaboration, and the safety chip uses a formal monitor to make sure that the robot stays operating within hard limits. Trust and safety are seperate properties, but these two papers compliment each other to resolve the questions mentioned in the introduction. Robots deployed in the real world will need the ability to influence how much humans rely on them, and they will also need verifiable guarantees that this reliance is never misplaced.

---





## Discussion Questions and In-Class Clarifications
This section summarizes questions raised during the presentation and supplements the recorded responses with clarifications from the papers and the reporter team’s analysis.



### 1. Questions about "Planning with Trust"

#### 1.1. Why is trust modeled with a Gaussian distribution?

The presenters explained that trust is more complicated than just a single number, so a distribution better represents it as a probability rather than one fixed, discrete value.

The paper uses Gaussian distributions in two places: the trust dynamics model predicts the next trust level as a Gaussian around a linear function of the current trust, and the questionnaire score is treated as a noisy reading of true trust. Since the robot never observes trust directly, a distribution lets it reason about a range of possible trust levels and allows for uncertainty across different human collaborators.

However, the Gaussian does not make trust multi-dimensional. Trust is still a single scalar value, and the distribution only expresses the *uncertainty* about it.

#### 1.2. How do the two models work together, and what is the prior over trust?

The presenters described the robot as starting from an initial trust value and updating it based on what the human does in his or her next action. More precisely, the robot tracks a belief, which is a probability distribution over the seven levels of trust there could be. The human decision model predicts how likely each action is at each trust level, so an observed action has the ability to shift the belief toward the trust level that best explains it.

The trust dynamics model then updates the belief to reflect how trust has evolved after the outcome of the action. The planner reasons over these future updates, which is how it decides that picking up the bottle first will be worth it to make the human less likely to interupt it when the robot decides to pick up the wine-glass later.

The paper does not report the initial belief used in the experiments. One natural choise might be to use the initial trust scores from the AMT data collection, but this is purely speculation.


### 2. Questions about "Plug in the Safety Chip"

#### 2.1. How can GPT-4 be used when it already has its own built-in constraints?

GTP-4's built-in safety comes from general alignment training, and it does not cover task-specific rules like the paper covered. The paper compares two ways of supplying these conditions: inside the prompt (NL constraints), and outside the prompt (safety chip). Inside the prompt is where GTP-4 would have to follow the rules itself, while outside the prompt has an external automaton reject unsafe actions.

The results of this paper favor keeping safety outside of the prompting, as NL constraints become less reliable as the number of constraints increases. This guarantee still is dependent on correctly translated LTL formulas, which is why expert verification is needed.

#### 2.2. When are the GPT models used in the pipeline?

GPT-4 is used for three language-based components:
- **NL-to-LTL translation:** converts safety requirements into formal constraints, which are then verified by human experts.
- **Action planning:** generates the robot's next action based on the task and execution history.
- **Violation explanation:** Generates natural-language feedback when the automaton rejects an action.

GPT-4 never decides whether an action is safe or not. The automaton performs this validation, which is what allows the guarantee to hold even when the planner is probabilistic.

### 3. Discussion questions

#### 3.1. Can trust models generalize to new robots, tasks, and users?

If a trust model is trained on table clearing, could it be applied to a hospital robot with different users, tasks, and risks? While the framework itself may be general, the learned parameters are tied to specific objects and outcomes of the table-clearing task. This means that these parameters would likely need to be relearned.

Trust findings themselves also appear to be fragile. A replication study found that a 40 person trust study did not replicate well with 140 to 200 people <d-cite key="ullman2021"></d-cite>. With 10 participants per condition in the real robot experiment, maybe the trust-POMDP's results should be replicated with a larger pool of participants and in other settings.

#### 3.2. If every specified rule is satisfied, is the robot safe?

Not necessarily, no. Even if the safety chip's safety rate was at 100%, that does not mean that ever situation possible has been modeled; especially for more complex environments. For example, a hospital robot that is told to "never enter restricted rooms" still satisfies all safety rules even when a patient suddenly crosses its path and nearly causes a collision. Unexpected physical hazards like this may need a seperate shielding in a motion-level safety layer.

#### 3.3. Is a safety shield enough?

The question has us consider two robots that satisfy the same safety requirements. Robot A is inspired by safety chip, and proposes an action, rejects it when it is unsafe, and replans. Robot B is inspired by SHARP, and predicts potential violations ahead of time, plans a safer alternative, and keeps a safety fallback. The question then asks which do we prefer to work with?

Both robots are technically equally safe, but Robot B appears to be the better collaborator because it avoids unsafe actions at the very start, while a robot that has to stop, check, and change its plan may seem less capable, even if both execute safe actions. However, Robot A does seem simpler to verify and can be attached to an existing agent without retraining. To truly decide which is better, other measures than safety need to be observed.

#### 3.4. How should a robot respond when trust conflicts with safety?

The question has us consider a situation where during an urgent task, the human says: "Take the shortcut. I trust you," but the shortcut crosses a prohibited area. This situation does seem to reflect overtrust, as the human's trust exceeds what the situation may justify, especially because it is an urgent circumstance. 

If the goal is to avoid losing trust, the robot should explain which rule would be broken if it proceeded with this shortcut and offer the fastest *safe* alternative instead. If there is a case or condition where the robot is allowed to cross into the restricted area, then the constraint should be updated to reflect this and verified by an expert.


