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
Large language models (LLMs) are exposed to multiple different semantic contexts through its vast pretraining dataset, enabling these models to gain an extensive knowledge base of the real world. Thus, a central question is how can an autonomous agent use an LLM's knowledge to carry out multi-step instructions in a world that it must interact with? Ideally, the LLM's advanced understanding of the world would greatly benefit autonomous agents, such as using common sense to find the desired object or selecting the correct next action.

However, there two major challenges when integrating LLMs into autonomous agents:
- LLM responses are conversational, so they cannot be directly used as actions for the robot to perform. Furthermore, these models do not have access to the available actions of the robot, resulting in semantically correct but unfeasible instructions.
- LLMs sequentially predict the best immediate action without considering the results of alternative actions. This can result in failures compounding if the LLM chooses the wrong action early on. This issue is exacerbated when there is a vast number of possibilities in the search space.

This lecture explores two different approaches to address the challenges:
- **Do As I Can, Not As I Say: Grounding Language in Robotic Affordances:** Rather than directly allowing the LLM to provide the next action, SayCan introduces a learned value function that computes the affordance score. This score represents which actions are feasible given the robot's abilities. Then, combining the LLM and the affordance score enables the robot to select the skill that is not only useful towards the goal but also attainable.
- **Large Language Models as Commonsense Knowledge for Large-Scale Task Planning:** To address the second challenge, LLM-MCTS combines the LLM's commonsense knowledge with Monte Carlo Tree Search (MCTS) to search for the best future. The LLM is used in two aspects. First, the model is used to construct a world model, acting as the initial belief of where objects are possibly located. Second, when the agent is working towards a task goal, MCTS is used to search for the best next action, using the LLM to bias the search towards the most likely actions. This narrows the search space to only explore the futures that could lead to successful outcomes.
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
