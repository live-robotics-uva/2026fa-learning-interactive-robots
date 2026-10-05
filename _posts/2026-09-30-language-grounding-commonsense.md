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
Large language models (LLMs) are exposed to multiple different semantic context through its vast pretraining dataset, enabling these models to gain an extensive knowledge base of the real world. However, these models cannot be directly used in autonomous agents to complete real-world tasks as they lack the ability to understand and interact with the physical environment; real-world understanding does not automatically translate into actionable items for the agent. Thus, a key question is how can an autonomous agent use an LLM's knowledge to carry out multi-step instructions in a world that it must interact with?

To answer this question, this lecture explores two different approaches to integrate LLMs into autonomous agents:
- **Do As I Can, Not As I Say: Grounding Language in Robotic Affordances:** One approach is to have the LLM select the agent's next action. However, as mentioned earlier, the LLM does not consider the physical constraints of the agent nor does it provide concrete actions to achieve the task goal. Thus, SayCan applies a value function to constrain the LLM's natural language commands to skills available to the robot. This enables the robot to select the skill that is not only useful towards the goal and but also attainable. 
- **Large Language Models as Commonsense Knowledge for Large-Scale Task Planning:** Another approach is to exploit the LLM's world knowledge to narrow the search space of possible actions. In the real world, there are various objects the agent can interact with and multiple places for it to search through, resulting in an exponential number of possible actions. Thus, the agent can use the LLM's knowledge to reason about the physical world, which guides the planner to select the best action based on commonsense. LLM-MCTS incorporates an LLM to provide a commonsense initial belief to start with, as well as suggesting a potential action plan to for the robot to take.
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
