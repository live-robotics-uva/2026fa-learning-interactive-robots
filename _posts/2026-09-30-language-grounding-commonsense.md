---
layout: distill
title: "Language Grounding & Commonsense"
description: "Grounding natural language commands into physical environments, spatial relationships, and affordance-aware commonsense reasoning."
date: 2026-09-30
future: true
htmlwidgets: true
ready: true

# Reporter team authors
authors:
  - name: "Lainey Gordon"
  - name: "William Smith"
  - name: "Wendy Zheng"
  - name: "Andrew Parkinson"

bibliography: 2026-09-30-language-grounding-commonsense.bib

toc:
  - name: "Introduction"
  - name: "Language Grounding"
  - name: "Commonsense Reasoning"
  - name: "Applications and Limitations"
    subsections:
      - name: "Applications"
      - name: "Limitations"
  - name: "Questions and Answers"
    subsections:
      - name: "Language Grounding Questions"
      - name: "Commonsense Reasoning Questions"
---

## Introduction {#introduction}

This lecture report covers the **Language Grounding & Commonsense** session in *Learning for Interactive Robots (CS 6501, Fall 2026)* at the University of Virginia.

> **Topic Overview**: Grounding natural language commands into physical environments, spatial relationships, and affordance-aware commonsense reasoning.

---

## Language Grounding {#language-grounding}

This lecture report covers the **Language Grounding & Commonsense** session in *Learning for Interactive Robots (CS 6501, Fall 2026)* at the University of Virginia.

> **Topic Overview**: Grounding natural language commands into physical environments, spatial relationships, and affordance-aware commonsense reasoning.

---

## Commonsense Reasoning {#commonsense-reasoning}

Given the wealth of commonsense knowledge encoded by Large language models, this next work explores the usefulness of LLMs as a policy versus a world model, proposing LLM-MCTS – an architecture that leverages this knowledge in both the model building and as a search heuristic in action selection. 

### LLM as a Policy vs. World Model

**Motivating Example**: An autonomous robot butler in a household environment. Consider asking the robot to put fruits in the fridge. Our commonsense reasoning allows us to narrow the vast search space of movable items and locations to likely considerations – the kitchen counter or cupboard as opposed to the bedroom closet. 

This work defines two means of utilizing an LLM in task planning:
- **L-Policy**: Given the history of past actions and observations, treat the LLM as a policy and query it directly for the next actions.
- **L-Model**: Use LLM’s commonsense knowledge to build a world model and apply a planning algorithm to the model.

L-Policy shows limitations regarding generalization, while L-Model performance depends on world model accuracy and planning algorithm efficiency. This paper combines the ideas of both to outperform either alone. 

### Problem Setup

**Task Environment**: VirtualHome, a household activity simulation platform, evaluated across 800 randomly generated large-scale, partially observable object rearrangement tasks.

**Formulation**:The problem is modeled as a Partially Observable Markov Decision Process (POMDP):

$$
\left(S, A, \Omega, T, O, R, \gamma \right)
$$

- State Space ($$S$$): positions of the robot, movable items, containers
- Action Space ($$A$$): pick, place, open, close, move
- Observation Space ($$\Omega$$): robot only able to see object/container positions within its room or an opened container at its location
- Transition Function ($$T$$): assumed as given and deterministic
- Observation Function ($$O$$): provides partial information about a state
- Reward Function: ($$R$$): reward for achieving the desire item arrangement
- Discount Factor ($$\gamma$$): discount factor on future rewards

The robot acts on the history of past observations and actions $$h_t = \left(o_0, a_0, o_1, a_1, \ldots, o_{t-1}, a_{t-1}  \right)$$, with the goal of maximizing an expected cumulative reward: 

$$
\pi^*\left(h_t\right) = \arg\max_{a \in A} \mathbb{E} [\sum_{i=0}^\infty \gamma^i R \left( s_{t+i}, a_{t+i} | a_t = a\right)].
$$

**Challenges**: long horizon planning, vast action space &rarr; exponentially large search tree

### LLM as a Commonsense World Model

The approach uses an LLM's commonsense knowledge to generate initial beliefs over object locations thus prioritizing search to more appropriate locations. Prompting details are as follows: Given expert actions and observations in similar environments, the LLMs are prompted to sample object positions $$M$$ times. Each time, according to a fixed prompt, it is asked to predict the position of an object. The response is then encoded and mapped to objects in the dataset. The sampled answers are counted and normalized to form a probability distribution over each object's location. 

Beliefs are maintained in object-centric graphs where the abstract-level relationships are the edges connecting objects (nodes). 

### LLM as a Heuristic Policy

The approach also uses the LLM as a policy; however, its role is specifically to guide action selection in the PUCT process. The LLM is sampled $$M$$ times for actions to take ($$\alpha_i$$), given the prompt and trajectory history ($$h$$), and the empirical policy distribution is thus formulated as such:

$$
\hat{\pi}\left(a | h \right) = \lambda \frac{1}{|A|} + (1 - \lambda)\text{Softmax}\{\sum_{i=1}^M \text{CosineSim}\left(\alpha_i, a \right) - \eta\},
$$

where $$\eta$$ is the average cosine similarity value and $$\lambda$$ is a hyperparameter adding randomness such that the search is not entirely reliant on the LLM suggestion. 

### Integration with Monte-Carlo Tree Search (MCTS)

{% include figure.liquid path="assets/img/2026-09-30-language-grounding-commonsense/r1-p2-arch.png"
class="img-fluid rounded z-depth-1"
caption="Overview of LLM-MCTS. For each simulation in the MCTS, sample from the commonsense belief to obtain an initial state of the world and use the LLM as heuristics to guide the trajectory to promising parts of the search tree."
%}

In alignment with the described framework, an MCTS simulation works as follows \[Ref in Alg 1]: 

1. Sample a state $$s$$ from the belief $$b(s)$$. \[Line 4]
2. Select an action according to $$a^*$$, considering the $$Q$$ value, visit counts, and LLM policy. \[Line 29]
3. MCTS expansion and random rollout returns a reward estimate. \[Line 14-17] 
4. Backpropagate the accumulated rewards to update each node's estimated $$Q$$ value. \[Line 32-35]
5. After $$N$$ simulations, the output is selected according to the highest $$Q$$ value. \[Line 3-8]
6. Execute action, obseve, update belief.

{% include figure.liquid path="assets/img/2026-09-30-language-grounding-commonsense/r1-p2-alg1.png"
class="img-fluid rounded z-depth-1"
%}

### Results

#### Setup
Data was generated from 2000 tasks with randomly initialized scenes and expert trajectories, and the framework was evaluated on 800 tasks in VirtualHome. Task types included *Simple* (rearrange one item from same distribution as dataset), *Comp.* (composition of simple tasks / rearrange multiple items), *Novel Simple* (tasks with seen items in novel task descriptions), and *NovelComp(2)* and *NovelComp(3)* (seen items in novel compositional task descriptions).  

**Success** is defined as completing the tasks within 30 steps, where completion is when all requirements of the object positions are satisfied.

**Baselines** include UCT <d-cite key="kocsis2006bandit"></d-cite> (planning without commonsense knowledge with ground-truth reward function), Finetuned GPT2 <d-cite key="li2022pretrained"></d-cite> (trained on 10,000 trajectories from training dataset), and GPT3.5 Policy <d-cite key="huang2022language"></d-cite> (LLM used as the policy only / no MCTS).

#### Results

{% include figure.liquid path="assets/img/2026-09-30-language-grounding-commonsense/r1-p2-table1.png"
class="img-fluid rounded z-depth-1"
%}

- MCTS without LLM (UCT) fails due to intractability – poor model and huge search tree.
- All other methods do reasonably well on Simple tasks, but the LLM-MCTS well-outperforms as tasks complexify.
- Fine-tuning compromises generalizability, and long-horizon planning introduces error accumulation which may not be included in prompt examples; MCTS encourages exploration.

**Ablation**: To evaluate individual component contributions, the authors ran an ablation study:

{% include figure.liquid path="assets/img/2026-09-30-language-grounding-commonsense/r1-p2-table2.png"
class="img-fluid rounded z-depth-1"
%}

- *No heuristic policy*: 0% in all cases; cannot efficiently conduct search for large-scale planning tasks.
- *Uniform state prior*: incorrect world models compromise search performance.
- *Fully observable*: marginal improvement over practical counterpart without full-observability.

### Discussion

**When is using an LLM as a model better than as a policy?** Minimum description length (MDL) principle / Occam’s Razor: 
> **If two methods fit the training data well, choose the method that has a shorter description.**

*Takeaway*: When the world is simpler to describe than the behavior, use the LLM as a world model and use a planner for reasoning, and vice versa. 

---

## Applications and Limitations {#applications-and-limitations}

Given the capabilities of language grounding and commonsense reasoning, what are the current applications and limitations of these methods?

### Applications {#applications}

Language grounding and commonsense reasoning serve a similar purpose when applied to robotic systems. Both aim to increase the reasoning capabilities of the system and increase the complexity of tasks and environment. Language grounding increases the physical knowledge provided to an LLM by incorporating observations, state, feasibility, etc. into the prompt so that the LLM will reason within the constraints. Commonsense reasoning allows for basic knowledge to be contained straight in the method without explicitly configuring all different combinations of tasks, scenarios, environments, etc. and the relationships between them. These methods can be applied to various system solutions but are most commonly applied to generalist systems because they require robust reasoning that can handle higher variances and uncertainties for its inputs.

A very common and popular problem application is household robotics, where a robot is assisting or completing a task for a human user. Given the variety of house layouts and potential tasks it is reasonably to use language grounding to ensure proposed subtasks are relevant and achievable<d-cite key="liu_grounding_2023"></d-cite> and commonsense reasoning to clear up ambiguity of task descriptions<d-cite key="kwon_toward_2024"></d-cite>.

{% include figure.liquid
   path="assets/img/2026-09-30-language-grounding-commonsense/grounded-commonsense.png"
   class="img-fluid rounded z-depth-1"
   caption="Figure 1: A household task of cleaning up a desk iterates between a VLM and LLM to infer the true task by reducing task ambiguity with self prompting and active perception. This allows an LLM to determine what subtasks should be completed to achieve the direct task (Kwon et al., 2024)."
%}

Without language grounding or commonsense reasoning, every task would have to be meticulously configured to make the robotic system robust to different layouts and terminologies. It would not be feasible to set up a generalist policy using hand configured tasks with the level of detail afforded by leveraging grounding and commonsense reasoning.

### Limitations {#limitations}

The benefits and advanced capabilities given by incorporating language grounding and commonsense reasoning into robotic systems do not come without risks and costs. Each system is unique and therefore subject to its own specific set of limitations but broadly there are three main limitations to keep in mind when leveraging these methods. *These limitations are not guaranteed to occur but can be prevalent and should be considered when determining total effectiveness.*

1. *Lack of explainability:* All methods that make use of machine learning, specifically deep learning, are just black box functions with no explainability about the results. Explainability is important for identifying why a predictions was made so that feedback can be used to correct the inputs and influence the next predictions to better align with the desired output. This is particularly important for tasks that require long horizon or high precision reasoning where small mistakes lead to costly errors. Since the errors are not explainable, then the results are less controllable and the system must trust all of the responses from the models. This open loop methodology drastically limits the task complexity a system can complete in a robust and trustful manner.
2. *Possibility of hallucinations:* When using an LLM, either with language grounding or for commonsense, to provide vital reasoning capabilities, any hallucinations render the prediction fundamentally incorrect. Since the LLM's are typically incorporated into robotic systems to handle environmental uncertainty, it is difficult to define rules to help verify the efficacy of an LLM response. With language grounding, the best case scenario for a hallucinated prediction is simply predicting the incorrect position of a target object leading to increased search time. Unfortunately, if the hallucination is bad enough the task could completely fail due to deadlocks<d-cite key="ahn_saycan_2022"></d-cite> or collision.
3. *Higher latency:* Language grounding and commonsense reasoning typically rely on combining results from different models and methods to share bolster the strengths while reducing the weaknesses. This means that there is typically more compute required to run the proposed architecture since there are more components and leads to an increased risk of high latency. There are architectural ways to reduce the added latency if the system is carefully considered and designed. LLM-MCTS<d-cite key="zhao_llmmcts_2023"></d-cite> was able to reduce the latency compared to a vanilla MCTS due to the heuristics applied from the LLM during the search. Even though LLM-MCTS was able to reduce latency compared to the baseline, it still has a latency that is too high for real world deployment. Therefore, the accuracy boosts should be considered carefully after weighing the latency costs.

---

## Questions and Answers {#questions-and-answers}

This section provides common questions related to the presented topics. *If questions ask to take a side, we provide answers to support both sides.*

### Language Grounding Questions {#language-grounding-questions}

Questions and answers related to language grounding and SayCan<d-cite key="ahn_saycan_2022"></d-cite>.

#### Q: Do you think natural language is the right way to program robots?

*Natural language is preferred:* Natural language is a wonderful way to program and instruct robots to complete tasks for many reasons. First, it lowers the entry barrier for users to interact with the robot. Second, it allows for a larger, hopefully open, vocabulary so tasks can be specific without every combination being configured in the robot. It also increases the interactability of the robot so that humans can potentially do other tasks while instructing the robot of a desired task.

*Natural language is insufficient:* Using natural language for programming or instructing a robot can be quite risky due to the ambiguity and uncertainty in different interpretations of the same natural language. Human communication also relies on more than just natural language when communicating with another human, with most communication being non verbal<d-cite key="mehrabian_nonverbal_1972"></d-cite>. Therefore, it seems inefficient to have the robot follow natural language when it could be learning and understanding body language and intent prediction. It's not that language shouldn't be used for programming robots but it should be a small component used to help the robot estimate the preferences of the human.
  
#### Q: How should robots use the commonsense knowledge already contained in LLM's?

Commonsense knowledge is meant to expand the understanding of basic scenarios and situations without having specific configurations for all possible tasks and environments. At its simplest, this can be accomplished by prompting the LLM with the task, known environment, and current state. The response will consider all of the provided information and can help propose a high level plan for execution. Without commonsense knowledge from an LLM, many tasks will require explicit modeling decreasing the generalization of a robotic system.

#### Q: Does SayCan actually ground the LLM in the physical world or does it only constrain the LLM using an external ground model?

*LLM is constrained:* The physical relations come from the value functions that encode the ability for a particular action to be completed given the current state. At runtime, the LLM reasoning is completed before being constrained by the probabilities from the trained value model. Since the LLM has no new information relating to the task and current capabilities of the robot, it finishes its inferences without being physical grounded. The LLM inferences are used for correlating actions to task progression. The appropriate action is then chosen as the action that maximizes the combined score from the LLM and the value model:

$$
p_{\pi} = p(c_{\pi} \vert s, l_{\pi}) p(l_{\pi} \vert i, l_{\pi_{n-1}}, \ldots, l_{\pi_{0}}),
$$

where $p_{\pi}$ is the policy probability, $p(c_{\pi} \vert s, l_{\pi})$ is the probability of a skill being completed, $p(l_{\pi} \vert i, l_{\pi_{n-1}}, \ldots, l_{\pi_{0}})$ is the probability of a skill being useful for the task, $s$ is the current state and $l_{\pi_{n}}$ is the textual representation of the skill. There is indeed a combinatorial effect but purely just a maximum likelihood of the joint probability.

*LLM is grounded:* The grounding comes directly from the temporal-difference based reinforcement learning which learns the value function. Since the LLM and value functions loop over all of the possible actions, the physical grounding is enforced through brute force. The LLM will reason about every possible action and how it progresses the task. The reasoning is then weighted based on the feasibility of the task from the value function. The grounding is loosely coupled between the two models but it still provides appropriate restrictions on the action space.

### Commonsense Reasoning Questions {#commonsense-reasoning-questions}

Questions and answers related to commonsense reasoning and LLM-MCTS<d-cite key="zhao_llmmcts_2023"></d-cite>.

#### Q: Given the runtime gap, does the GPT-3.5-MCTS justify the added latency?

*Latency is too high:* Runtimes approaching 70 seconds is completely unacceptable and insufficient for a real world deployment of a robotic system. These latencies are reserved for offline planning methods and not suitable for realtime and reactive planning. Given that the whole point of leveraging the LLM is to react better to an undefined environment and increase adaptability, it heavily implies that the robot should be able to run this onboard in a reasonable time. In hindsight this paper seems to have made poor architectural choices by using such a large LLM and MCTS, both of which take a long time to run compared to modern vision language action models<d-cite key="black_pi0_2025"></d-cite> which leverage smaller backbone models with under 3B parameters<d-cite key="beyer_paligemma_2024"></d-cite> and run in realtime to with comparably complex tasks.

*Latency is justified:* Just based on the results shown in the paper, it is difficult to not justify the added latency since the experiments show significantly degraded performance (sometimes approaching zero) when deploying the baseline methods. In this case you would have to take the added latency for any hope of achieving the task. Given the experimental setup focusing on long horizon tasks in a household environment, the latency can probably be distributed throughout the mission so that once the initial step is predicted then the other predictions come in at what appears to be a reasonable time because the robot is thinking ahead while completing the current task. It would be interesting to see a distribution of what steps of the algorithm took the most time because they authors show that the GPT3.5 Policy and GPT2 Policy were the fastest and that the experiments with MCTS (GPT3.5-MCTS and UCT) were the slowest. This would allow us to decompose the problem to possibly determine how much of the latency is attributed to the task/environment space since it appears the initially appears MCTS is the bottleneck.

#### Q: LLM-MCTS uses the LLM to predict where objects probably are (grounding its knowledge into the environment). What happens to the whole system if that prediction is wrong?

Simply put, if the LLM predicts object locations incorrectly then the guiding heuristic for the MCTS will drive the search towards the predicted location. The robot would attempt to execute the chosen tasks. In the best case, the robot navigates to a location then discovers there is no way to retrieve the object and replans another location prediction. This just adds time to the task but still gets completed eventually. In the worst case, the robot navigates to where it believes the object to be and then when it replans it replans the same false belief. The robot will get stuck in this situation and not be able to recover. The authors showed examples where the approach would get stuck in these deadlocks and thus fail the task. It would be neat to see an extension where the approach allowed for the LLM to learn by keeping a richer context history so that it will not keep proposing the same incorrect belief over and over again. Maybe a deterministic algorithm could be used to detect these deadlocked states and then explicitly prompt the LLM with details about the currently deadlock and this would allow for a new prediction to be made.

#### Q: The LLM's commonsense suggestions are only used as a heuristic to guide search, not followed directly. Why might that be safer than just letting the LLM pick the action outright?

As everyone knows, LLM's have issues with hallucinating and there are currently no reliable ways to detect hallucinations for the given problem statement. This is a major issue if the LLM is given complete control over a robotic system because it could instruct the robot to perform actions that are dangerous to itself or others, like moving joints into collision regions. Compared to hallucinating object locations, hallucinating specific capabilities that are not achievable can be catastrophic. Therefore, the MCTS constrains the search space enough to prevent completely unforeseen actions from being selected.

#### Q: What is the insight behind minimum description length (MDL)?

The authors mention choosing between L-Model and L-Policy by utilizing the MDL principle. It is then shown with a few examples that a reasonable prediction for which model is optimal can be obtained from the problem formulation based on the information required to represent the problem in either form. MDL is a simple concept originating in information theory where the goal was to determine the minimum number of parameters required to represent a dynamical system<d-cite key="rissanen_modeling_1978"></d-cite>, even under the effect of noise and disturbances. This has been extended to statistical<d-cite key="grunwald_minimum_2019"></d-cite> and machine learning models<d-cite key="shwartz_understanding_2014"></d-cite> with proofs showing the loss function function of the entire dataset can be bounded by the loss function of the training data plus a function of the current model<d-cite key="shwartz_understanding_2014"></d-cite>:

$$
L_{\mathcal{D}}(h) \leq L_{\mathcal{S}}(h) + \sqrt{\frac{\lvert h \rvert + \ln{(2/\delta)}}{2m}},
$$

where $\mathcal{S} \sim \mathcal{D}^{m}$ is the training set sampled from the whole dataset, $m$ is the size of the dataset, $L_{\mathcal{D}}$ is the expect loss on the entire dataset, $L_{\mathcal{S}}$ is the loss on the training set, $\delta$ is the confidence parameter, and $h$ is the model with $\lvert h \rvert$ representing the number of parameters. This equation shows that two models with the same training loss but different descriptor lengths will have different upper bounds for the generalization loss directly due to the number of parameters since everything else in the equation is the same. Therefore, we should prefer models that are smaller to reduce overfitting and generalization errors.
