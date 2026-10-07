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
---

## Introduction {#introduction}

This lecture report covers the **Language Grounding & Commonsense** session in *Learning for Interactive Robots (CS 6501, Fall 2026)* at the University of Virginia.

> **Topic Overview**: Grounding natural language commands into physical environments, spatial relationships, and affordance-aware commonsense reasoning.

---

## Language Grounding {#language-grounding}

This lecture report covers the **Language Grounding & Commonsense** session in *Learning for Interactive Robots (CS 6501, Fall 2026)* at the University of Virginia.

> **Topic Overview**: Grounding natural language commands into physical environments, spatial relationships, and affordance-aware commonsense reasoning.

Large Language Models can be a powerful tool in translating a simple instruction or goal into a set of concrete steps a robot can take to achieve that goal. Through training on massive amounts of data they are able to identify ambiguous relationships and adapt to many scenarios. Despite their ability to understand deep semantic relationships and make inferences that aren't explicitly stated in the instruction they still lack knowledge of the specific environment a robot is or its abilities to complete any given subtask. 

### Do As I Can, Not As I Say: Grounding Language in Robotic Affordances <d-cite key="ahn_saycan_2022"></d-cite>

SayCan solves this problem with *Language Grounding*, which tells the LLM the abilities of the robot, its current state, and the state of the scene the robot is acting in. They break down the abilities of the robot into a list of predefined subtasks such as "pick up object", or "go to the sink". For each of these skills they use the LLM to estimate the probability that completing that task will increase progress towards goal, and they use an affordance function to get the probability the skill can be completed given the current state. This combined probability of a skill successfully making progress on the instruction is factorized as:

$$p(c_i|i,s,l_\pi) \propto p(c_\pi|s,l_\pi) p(l_\pi|i)$$

where $p(l_\pi|i)$ represents task-grounding from the LLM and $p(c_\pi|s,l_\pi)$ represents world-grounding from the affordance function.

The set of skills gives the LLM context on the robots abilities, and the affordance function gives the robot context on the environment. 

SayCan uses RL and BC to learn both the skills and the affordance function. The learning policies are trained using sparse rewards, where 1.0 is given for success, and 0.0 for failure. This means that the learned value function is the same as an affordance function. The LLM used was 540B parameter PaLM.

#### Execution 
Given the initial instruction, the set of skills the robot can perform, and the affordance function, SayCan evaluates each skill using the LLM to give a probability that skill progresses, and using the affordance function to give probability of completion. At each step it choses the skill with the highest combined probability, determining the optimal skill via:

$$\pi = \arg\max_{\pi \in \Pi} p(c_\pi|s,l_\pi) p(l_\pi|i)$$

and then the next skills are chosen with the new state of the robot and environment. This is repeated until the task is completed.

In order to handle a wider range of tasks such as negations, they introduced chain of though reasoning. They instructed to model to explain how it scores each of the skills which improves its reasoning and performance on instructions like "bring me a snack that isn't an apple". 

#### Explainability
By using a LLM to decide each step in natural language, they produce a plan that is extremely interpretable. 

Under an ablation study that swaps out the LLM for smaller versions of PaLM, or for FLAN they show that as the LLM improves, SayCan's performance increases without having to retrain the skills. For example, using the 137B FLAN model resulted in a 70% planning success rate and 61% execution success rate, whereas the 540B PaLM model achieved 84% planning and 74% execution. 

Not only can they improve the performance of SayCan by dropping in a new improved LLM, but if they need to add new skills for a different environment they can just add it into the LLM's prompt. The huge depth of knowledge the LLMs have been trained on allows for huge adaptability in tasks SayCan hasn't been initially designed for.

#### Results
PaLM-SayCan was able to achieve 84% planning success, and 74% execution success in the mock kitchen, with a 3% drop in planning and 14% drop in execution when testing in the real kitchen. It struggles the most with long horizon tasks that would take many intermediate steps, but the LLM will often terminate early. Ablations also proved the necessity of both systems: removing the affordance value functions (No VF) dropped planning success to 67%, while removing the language model entirely (BC NL) resulted in a 0% success rate across all tasks.




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

Questions in the presentation:

- Language Grounding
  - Do you think natural language is the right way to program robots?
  - How should robots use the commonsense knowledge already contained in LLM's?
  - Does SayCan actually ground the LLM in the physical world or does it only constrain the LLM using an external ground model?
- Commonsense Reasoning
  - Given the runtime gap, does the GPT-3.5-MCTS justify the added latency?
  - LLM-MCTS uses the LLM to predict where objects probably are (grounding its knowledge into the environment). What happens to the whole system if that prediction is wrong?
  - The LLM's commonsense suggestions are only used as a heuristic to guide search, not followed directly. Why might that be safer than just letting the LLM pick the action outright?

Questions asked in lecture:

- What is the insight behind minimum description length (MDL)?
