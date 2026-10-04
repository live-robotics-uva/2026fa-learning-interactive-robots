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

This lecture report covers the **Language Grounding & Commonsense** session in *Learning for Interactive Robots (CS 6501, Fall 2026)* at the University of Virginia.

> **Topic Overview**: Grounding natural language commands into physical environments, spatial relationships, and affordance-aware commonsense reasoning.

---

## Applications and Limitations {#applications-and-limitations}

This lecture report covers the **Language Grounding & Commonsense** session in *Learning for Interactive Robots (CS 6501, Fall 2026)* at the University of Virginia.

> **Topic Overview**: Grounding natural language commands into physical environments, spatial relationships, and affordance-aware commonsense reasoning.
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
