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
