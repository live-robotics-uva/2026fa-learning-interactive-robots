---
layout: distill
title: "Human Models"
description: "Computational human models: Boltzmann rationality, bounded rationality, Theory of Mind, and belief modeling for human-robot interaction."
date: 2026-10-21
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
  
bibliography: 2026-10-21-human-models.bib

toc:
  - name: "Introduction"
    subsections:
    - name: "Boltzmann Rationality"
    - name: "Inferring Intent"
    - name: "Bayesian Theory of Mind"
    - name: "Markov Decision Process(MDP)"
    - name: "Partially Observable Markov Decision Process(POMDP)"
  - name: "LESS is More: Rethinking Probabilistic Models of Human Behavior"
    subsections:
        - name: "Motivation"
        - name: "Background"
        - name: "Less Model"
        - name: "Less as Human Decision Model"
#          subsections:
#            - name: "Design"
#            - name: "Results"
        - name: "LESS for Inference"
#          subsubsections:
#            - name: "Boltzmann & Less Comparison"
#            - name: "7-DOF Manipulator Demonstration"
        - name: "Limitation"
    
  - name: "Large Language Models as Zero-Shot Human Models for Human-Robot Interaction"
    subsections:
        - name: "Motivation"
        - name: "Problem Definition"
        - name: "Experiment Setup"
        - name: "Experiment Result"
        - name: "LLMs on Tasks"
        - name: "LLMs Sensitivity to Prompt Structure"
        - name: "Case Studies: Planning"
#          subsections:
#            - name: "Table Clearing"
#            - name: "Utensil Passing"
        - name: "Conclusion"
  - name: "Question and Answer"
---

## Introduction
> **Topic Overview**: Computational human models: Boltzmann rationality, bounded rationality, Theory of Mind, and belief modeling for human-robot interaction.


This lecture report covers the **Human Models** session in *Learning for Interactive Robots (CS 6501, Fall 2026)* at the University of Virginia. We explored how robots can understand, predict, and respond to human behavior when sharing an environment by reviewing two papers:
* **LESS is More: Rethinking Probabilistic Models of Human Behavior**: 

Traditional Boltzmann models can assign excessive probability to groups containing many nearly identical trajectories. LESS corrects this by accounting for trajectory similarity, improving human behavior modeling and certain inference tasks.
* **Large Language Models as Zero-Shot Human Models for Human-Robot Interaction**: 

Investigates whether pretrained LLMs can predict human judgments, trust, and social responses without task-specific training. Results show promising performance, but limitations remain in spatial and physical reasoning.

### Why do we need human model? 
Imagine a robot navigating a hallway with several people. The robot knows its destination and can detect obstacles, but it doesn't necessarily know where people intend to go.
The main challenge is that human behavior is uncertain. Humans can change direction, pursue different goals, and react to the robot itself.
A human model helps a robot:
* Infer intent: Understand what a person is trying to accomplish.
* Predict behavior: Anticipate the person's future actions.
* Plan interactions: Choose safe and effective actions based on those predictions.

### How do we model human behavior?
The lecture introduces several approaches:
* **Boltzmann Rationality**: Approxinmating human decision with a likelihood distribution
* **Bayesian Theory of Mind**: Reasoning about beliefs of others with Bayesian updates
* **Markov Decision Process (MDP)**
* **Partially Observable Markov Decision Processes (POMDP)**




---
## LESS is More: Rethinking Probabilistic Models of Human Behavior

## Large Language Models as Zero-Shot Human Models for Human-Robot Interaction

## Presentation Q&A:
### LESS is More: Rethinking Probabilistic Models of Human Behavior
* The experiments show that LESS sometimes performs worse than Boltzmann. Then, does that indicate that we shouldn't consider trajectory similarity when choosing an action? In the real world, when should we treat similar choices as distinct options?
* The authors used predefined feature vectors for computing trajectory similarity. Does this limit the performance of LESS? If so, how can we learn those features or extract them automatically?

### LLMs as Zero-Shot Human Models for Human-Robot Interaction
* Since LLMs are only used to predict human behavior, does it matter that these models perform poorly on spatial and numerical reasoning?
* The authors observed that LLM performance is sensitive to prompt design. Would this limitation make LLMs ineffective as human models?
* The authors used very simple tasks to test LLMs as human models. But, in the real world, would LLMs be sufficient to approximate humans?
* When interacting with humans, some people will do the exact opposite of what's expected, ex: intervene for a whisk but not a knife, seemingly just to be contrarian. How can an LLM or Robot account for this?


### Report Status

This report is currently being prepared and will be updated by the student reporter team following the lecture session and classroom discussion. Please check back soon!
