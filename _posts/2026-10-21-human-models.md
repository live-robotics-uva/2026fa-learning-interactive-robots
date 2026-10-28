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
#    subsections:
#    - name: "Boltzmann Rationality"
#    - name: "Inferring Intent"
#    - name: "Bayesian Theory of Mind"
#    - name: "Markov Decision Process(MDP)"
#    - name: "Partially Observable Markov Decision Process(POMDP)"
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
  - name: "Question"
  - name: "Reference"
---

## Introduction

> **Topic Overview**: Computational human models: Boltzmann rationality, bounded rationality, Theory of Mind, and belief modeling for human-robot interaction.

This lecture report covers the **Human Models** session in *Learning for Interactive Robots (CS 6501, Fall 2026)* at the University of Virginia. We explored how robots can understand, predict, and respond to human behavior when sharing an environment by reviewing two papers:

-   **LESS is More: Rethinking Probabilistic Models of Human Behavior**<d-cite key="bobu2020less"></d-cite>:
Traditional Boltzmann models can assign excessive probability to groups containing many nearly identical trajectories. LESS corrects this by accounting for trajectory similarity, improving human behavior modeling and certain inference tasks.

-   **Large Language Models as Zero-Shot Human Models for Human-Robot Interaction**<d-cite key="zhang2023large"></d-cite>:Investigates whether pretrained LLMs can predict human judgments, trust, and social responses without task-specific training. Results show promising performance, but limitations remain in spatial and physical reasoning.

### Why do we need human model?

Imagine a robot navigating a hallway with several people. The robot knows its destination and can detect obstacles, but it doesn't necessarily know where people intend to go. The main challenge is that human behavior is uncertain. Humans can change direction, pursue different goals, and react to the robot itself. A human model helps a robot:

-   Infer intent: Understand what a person is trying to accomplish.
-   Predict behavior: Anticipate the person's future actions.
-   Plan interactions: Choose safe and effective actions based on those predictions.

### How do we model human behavior?

The lecture introduces several approaches:

-   **Boltzmann Rationality**: Approxinmating human decision with a likelihood distribution
-   **Bayesian Theory of Mind**: Reasoning about beliefs of others with Bayesian updates
-   **Markov Decision Process (MDP)**: Modeling sequential decision-making based on the current state, actions, transitions, and rewards.(agent always knows the exact, true state)
-   **Partially Observable Markov Decision Processes (POMDP)**: Modeling decision-making under uncertainty by maintaining and updating a belief state from observations.(agent doesn’t know the current state)

---

## LESS is More: Rethinking Probabilistic Models of Human Behavior

### Motivation

Human-robot collaboration requires a model of human behavior for both inference and prediction. For inference, the robot uses the model to recover human goals and preferences from observed behavior. For prediction, the robot uses the model to anticipate future human behavior and coordinate its own actions.

A standard choice is the **Boltzmann noisily-rational decision model**, which assigns higher likelihood to trajectories with higher reward. It was originally developed for decisions among discrete and distinct options. However, human trajectories lie in a continuous space with a continuum of alternatives that vary in their similarity to one another. Because the Boltzmann model does not explicitly account for these similarities, adding nearly identical, equally rewarding trajectories can increase their group's total probability even though they represent essentially the same behavior. **LESS (Limiting Errors due to Similar Selections)** addresses this issue by accounting for both reward and similarity among available trajectories. <d-cite key="bobu2020less"></d-cite>

### Background

Real-world trajectory spaces are full of near-duplicates. Consider two equally rewarding trajectories to a goal, one going left and one going right. Boltzmann assigns each probability $$1/2$$. Adding two trajectories similar to the right-hand trajectory, with all four rewards equal, changes each probability to $$1/4$$. The total probability of going right becomes $$3/4$$, even though the added options largely repeat an existing route. However, the idealized LESS correction retains a probability of $$1/2$$ on the left and reduces the probability of each similar right-hand trajectory to $$1/6$$.

{% include figure.liquid
   path="assets/img/2026-10-21-human-models/less-figure-1-full.png"
   class="img-fluid rounded z-depth-1"
   alt="Top: two equally rewarding routes receive equal probability; adding similar right-hand routes changes the Boltzmann probabilities, while the idealized LESS correction preserves the balance between groups. Bottom: a person providing a demonstration with a 7-DOF robotic arm, with trajectories shown in an inset."
   caption="Bobu et al. (2020), Figure 1. Boltzmann reduces the left probability from 1/2 to 1/4; the idealized LESS correction retains a probability of \(1/2\) on the left and reduces the probability of each similar right-hand trajectory to \(1/6\). The bottom panel shows a user demonstration with a 7-DOF robotic arm."
   zoomable=true
   avoid_scaling=true
%}

The choice rule comes from Luce's choice axiom, which assigns each option a probability proportional to its desirability. For a finite option set $$O$$, let $$v(o)>0$$ denote the desirability of option $$o\in O$$. The selection probability is

$$
P(o)=\frac{v(o)}{\sum_{\bar o\in O}v(\bar o)}.
$$

Letting $$v(o)=\exp(R(o))$$, we obtain the Luce-Shepard choice rule. When the options are trajectories $$\xi\in\Xi$$ (where $$\Xi$$ is the feasible trajectory space), this becomes the Boltzmann model of noisily rational behavior. With feature vector $$\phi(\xi)$$ and reward $$R(\phi(\xi))$$, its continuous density is

$$
p_{\mathrm B}(\xi)
=\frac{\exp(R(\phi(\xi)))}
{\int_{\Xi}\exp(R(\phi(\bar\xi)))\,d\bar\xi}.
$$

For a finite choice set, the normalizing integral becomes a sum. Each alternative trajectory contributes separately according to its reward, so adding duplicates changes selection probabilities. Extensions of Luce's model address this problem by grouping identical or similar options. <d-cite key="bobu2020less"></d-cite>

One such method is the Attribute Rule proposed by Gul et al. It treats options as bundles of attributes. Choice proceeds in two stages: first choose an attribute $$x$$ using Luce's rule, then choose an option $$o$$ with that attribute, weighted by its intensity $$s(x,o)$$.

LESS takes advantage of the Attribute Rule's handling of duplicates while extending it to trajectories with continuous-valued features. <d-cite key="bobu2020less"></d-cite>

### Less Model

LESS modifies the Attribute Rule by treating the feature vectors of trajectories as attributes. Trajectories with the same features are treated as a single option, and the group's probability is distributed equally among them. If every trajectory has a unique feature vector, this exact-matching construction recovers the Boltzmann model.

However, real-world trajectories can be very similar without being exactly the same. Therefore, LESS introduces a nonnegative similarity function $$s(\phi(\xi),\bar\xi)$$. This function is symmetric between trajectories and reaches its maximum when a trajectory is compared with itself. As a result, the model adjusts reward-based choice based on aggregate similarity:

$$
\begin{aligned}
D(\xi)&=\int_{\Xi}s(\phi(\xi),\bar\xi)\,d\bar\xi,\\
p_{\mathrm L}(\xi)&=\frac{1}{Z_{\mathrm L}}
\frac{\exp(R(\phi(\xi)))}{D(\xi)}.
\end{aligned}
$$

In the equation, $$D(\xi)$$ measures aggregate similarity to all available trajectories, and $$Z_{\mathrm L}$$ normalizes the density. High reward still increases a trajectory's weight, but similar alternatives reduce its individual weight. In the idealized left-right example, the right-hand trajectories share their group's probability. For general trajectories, the adjustment depends on their feature similarities. <d-cite key="bobu2020less"></d-cite>

The paper uses **Kernel Density Estimation (KDE)** with a radial basis function (RBF) kernel to measure feature-space crowding:

$$
s(x,\bar\xi)
=\frac{1}{\sigma\sqrt{2\pi}}
\exp\left(-\frac{\|x-\phi(\bar\xi)\|^2}{2\sigma^2}\right),
\qquad \sigma>0.
$$

In this equation, $$x$$ is a feature vector, and $$\sigma$$ is the bandwidth, which controls how broadly similarity extends. Larger bandwidths make trajectories appear more similar. The paper fits the bandwidth through a KDE likelihood objective on sampled trajectories and approximates aggregate similarity by summing kernel values over a finite trajectory set.

Given a belief $$b(\theta)$$ over the reward parameters $$\theta\in\Theta$$, LESS predicts human trajectories by marginalizing over those parameters:

$$
p_{\mathrm{pred}}(\xi)
=\int_{\Theta}b(\theta)p_{\mathrm L}(\xi\mid\theta)\,d\theta.
$$

After observing a human trajectory $$\xi$$, Bayesian inference updates the belief over $$\theta$$:

$$
b'(\theta)
=\frac{b(\theta)p_{\mathrm L}(\xi\mid\theta)}
{\int_{\Theta}b(\bar\theta)p_{\mathrm L}(\xi\mid\bar\theta)\,d\bar\theta}.
$$

For tractability, the paper uses a discretized set of reward parameters and finite trajectory samples. <d-cite key="bobu2020less"></d-cite>

### Less as Human Decision Model

Before exploring whether LESS helps a robot, the authors explore whether LESS is a better description of how people actually make decisions than the Boltzmann model? They therefore designed and conducted the following experiments.

#### Design

It is difficult to test a decision model on real people, since the authors hardly derive one's real reward function and hidden preferences. They avoid this problem with a browser-based gridworld study made of two trials, where participants make navigation choices on a grid world:

- **Control trial (a):** one path is on the left, and one path is on the right, and both paths have equal length. Together with an attention test, this trial measures the baseline preference $\lambda$: the baseline choice ratio gives a relative preference between left and right without needing to know the exact reward formula.
- **Experimental trial (b):** Two additional near-copies of the optimal right path are added to the same layout as (a). The right side now offers three nearly identical paths (labeled R1, R2, and R3).

It is the same duplicates problem from the Background section. Under Boltzmann, the near-copies inflate the denominator, while LESS treats the right cluster as roughly a single option. Therefore, Boltzmann predicts a sharp drop in the probability of choosing the left path, while LESS predicts minimal change.

#### Results

In the control trial, 47.5% of participants chose the left path. In the experimental trial, 41% chose the left path, while Boltzmann predicted 23% and LESS predicted 47.5%. The predictions are compared with the observed choices using a chi-square test:

| Source | P(left) in experimental trial | Chi-square test against human choices |
|---|---:|---|
| Observed | 41% | — |
| Boltzmann prediction | 23% | χ² = 6.27, *p* < 0.05 (significantly different) |
| LESS prediction | 47.5% | χ² = 0.72, *p* = 0.4 (no significant difference) |

Boltzmann's prediction is significantly different from the human choices, while LESS shows no significant difference from them.

**Takeaway:** LESS is a significantly better fit for human decision-making.

### LESS for Inference

The user study shows that LESS is a better model for predicting human choices. The next question is whether a better behavior model also leads to better inference.

#### Inference with a Behavior Model

The robot keeps a belief $b(\theta)$ over the reward parameters $\theta \in \Theta$ and uses the behavior model $p(\xi \mid \theta)$ in two ways.

**Prediction.** Given the belief $b(\theta)$, marginalize over $\Theta$ to forecast human paths:

$$
p(\xi) = \int_{\Theta} b(\theta)\, p(\xi \mid \theta)\, d\theta.
$$

**Bayesian inference.** After observing a human path $\xi$, update the belief over $\theta$:

$$
b'(\theta) = \frac{b(\theta)\, p(\xi \mid \theta)}{\int_{\Theta} b(\bar\theta)\, p(\xi \mid \bar\theta)\, d\bar\theta}.
$$

The authors conducted two experiments:

| Experiment | Setting | Manipulated factors | Metric | Main finding |
|---|---|---|---|---|
| Boltzmann & LESS comparison | Simulated gridworld with two objects | Sampling model × inference model; 8 rewards × 150 seeds | TrueMatch, TruePosterior | Inference is best when the inference model matches how the demonstrations were generated |
| 7-DOF manipulator | JACO arm, 12 users | Inference model × sample set size (5 sizes, 10 sets each) | KLAggregate | LESS is more consistent for single demonstrations, while Boltzmann is more consistent for batch inference |

#### Boltzmann & LESS Comparison

**Setup.** In a gridworld, a robot learns from human demonstrations about whether the human prefers to stay close to or far from two objects. 
- **Sampling model:** the simulated human demonstrations are generated by Boltzmann or LESS.
- **Inference model:** the robot infers $\theta$ with either Boltzmann or LESS.

Each condition is tested across $8$ reward settings and $150$ random seeds. The following metrics are used.
- **TrueMatch**: it tests whether the most possible reward of a robot  is close to the true reward.
- **TruePosterior**: it tests the amount of belief the robot places on the correct answer.

The results show that inference performance is significantly higher when the inference model matches the sampling model. Additionally, Boltzmann is sensitive to feature density. (1) In **high-density**, the robot **under-learns**. (2) In **sparse**, the robot **over-learns**.

LESS measures each trajectory by the inverse of its density, which prevents feature density from influencing inference strength. After LESS's modification, the density is nearly flat, so it does not controls how much the robot learns.

#### 7-DOF Manipulator Demonstration
**Real-world challenge.** In a real-world environment, the denominator is intractable, so trajectory sample sets are required. Since Boltzmann is sensitive to feature space density, its inference results may vary hugely across different trajectrory sample sets. This experiment therefore measures robustness across sample sets.

**Settings.** $12$ participants interact with a 7 degree of freedom (DOF) JACO arm on a household manipulation task.

**Metric.** KLAggregate is the metric used to measure robustness. Specifically, it measures inference consistency across different trajectory sample sets. 

**Results.**
- **Single demonstration:** LESS is more consistent ($S\geq 300$), and the two models converge with more $300$ set size.
- **Batch inference:** Boltzmann is more consistent when all $12$ demonstrations are leveraged to represent an ``average'' user.

### Limitation
The presentor pointed out three limitations of LESS:
- Similarity depends on pre-specified features. LESS measures similarity between the feature vectors of trajectories, and these features are pre-specified, so feature misspecification is a risk.
- Batch inference is less robust. In the 7-DOF manipulator demonstration, LESS is less consistent than Boltzmann for batch inference. It is also unclear whether it is also less accurate.
- Simplistic datasets of human choices; more realistic studies are complicated by the lack of access to ground truth.

## Large Language Models as Zero-Shot Human Models for Human-Robot Interaction

### Motivation
Human models enable robots to understand and reason about human behavior, allowing them to plan their actions while considering their potential impact on humans. However, existing methods face two main challenges:
- **Limited flexibility and scalability** of handcrafted human models due to strong assumptions.
- **High demand for human interaction data** in non-parametric models.

To address these challenges, this paper explores the potential of **Large Language Models (LLMs) to serve as zero-shot human models for Human-Robot Interaction (HRI).** The authors demonstrate that LLMs, pretrained on vast amounts of human-generated text, can effectively model human latent states and behaviors. However, LLMs struggle with tasks requiring spatial and numerical reasoning. Additionally, their performance is sensitive to prompt phrasing and structure.

To further explore the ability of LLMs, they integrate LLM-based human models into robot planning and evaluate them in two HRI scenarios: a table-clearing task and a utensil-passing task. The results show that LLM-based planning can mitigate human overtrust and improve task performance. Nonetheless, LLMs may need to be combined with low-level models that capture geometric and physical information.

{% include figure.liquid
   path="assets/img/2026-10-21-human-models/llm-human-model-figure-1-overview.png"
   class="img-fluid rounded z-depth-1"
   alt="Interaction history and the current state and robot action are textualized into a prompt. An LLM predicts a human action or state distribution, which a separate planner uses to obtain a robot policy."
   caption="LLM-based human modeling and planning workflow."
   zoomable=true
%}

<p style="text-align: center; font-size: 1.5em;"><strong>LLMs as zero-shot human models</strong></p>

### Problem Definition

Let $$s_t$$ denote the environment state at time $$t$$, where the environment contains $$N$$ agents (humans or robots), denoted as $$a^{(i)} \in \mathcal{A}, \quad i=1,\ldots,N$$. The interaction history is defined as

$$
h_t = \left\lbrace \left(s_k, \lbrace u_k^{(i)}\rbrace_i \right) \right\rbrace_{k=1}^{t-1} \in \mathcal{H}
$$

where $$u_k^{(i)}$$ represents the action of agent $$i$$ at time $$k$$.

The goal is to use LLMs to model a human agent's action or mental state, represented by $$z_t^H$$, such as trust in a robot or the decision to intervene. Specifically, the authors aim to approximate the conditional probability distribution:

$$
p(z_t^H \mid s_t,h_t).
$$

A textualization function $$f$$ transforms the environment state, interaction history, and set of agents $$\mathcal{A}$$ into a textual prompt:

$$
x_t = f(s_t,h_t,\mathcal{A})
$$

The LLM-based human model, denoted by $$p_l$$, then predicts the distribution of the target human property $$z_t^H$$ given the prompt $$x_t$$. The objective is to evaluate whether this predicted distribution can accurately approximate the true conditional distribution of human behavior or mental states:

$$
p_l(z_t^H\mid x_t)\approx p(z_t^H\mid s_t,h_t)
$$

### Experiment Setup

**Models:** The prediction experiments compare **DAVINCI** (`text-davinci-003`, 175B) and **T5** (`FLAN-T5-XXL`, 11B). DAVINCI is accessed through an API, while T5 runs locally. 

**Datasets:** This paper uses two HRI-related datasets and a human-human social interaction dataset. Each dataset has a different specialized baseline, trained or developed for that prediction problem. The datails of these datasets are shown below:

| Dataset | Prediction task | This dataset is for | Baseline model |
|---|---|---:|---|
| <span style="position: relative; display: inline-block;">MANNERS-DB<d-cite key="tjomsland2022manners"></d-cite></span> | Social appropriateness of a robot action, rated from 1 to 5 | Evaluating whether LLM-based human models can capture the effects of physical and spatial features in HRI | BNN-16CL2 |
| <span style="position: relative; display: inline-block;">Trust-Transfer<d-cite key="soh2018transfer"></d-cite></span> | Trust after observing robot demonstrations, rated from 1 to 7 | Evaluating whether LLMs can capture the effect of interaction history on a human's mental state | Gaussian Process and Neural Network (GPNN) |
| SocialIQA | Social commonsense reasoning with three answer choices | Evaluating LLM's ability to reason about human social interactions | Fine-tuned BERT-large |

**Prompts:** Prompts include task instructions, relevant environment information, interaction history where applicable, and a question about the human response. The LLMs receive textual descriptions rather than directly inspecting the scene images used in MANNERS-DB annotation. The examples of prompts in each dataset is shown below.

{% include figure.liquid
   path="assets/img/2026-10-21-human-models/llm-human-model-figure-2-datasets-prompts.png"
   class="img-fluid rounded z-depth-1"
   alt="Examples from MANNERS-DB, Trust-Transfer, and SocialIQA, showing each dataset scenario and the corresponding text prompt used to predict human judgments or responses."
   caption="Datasets and example prompts used in the prediction experiments."
   zoomable=true
%}

**Metrics:** The paper evaluates three aspects of a human model: the error in its predictions, agreement with human labels, and how well its predicted uncertainty matches the variation in human responses. These aspects are complementary: a model can predict the majority answer correctly while assigning an unrealistic probability distribution to the other answers. <d-cite key="zhang2023large"></d-cite>

**1. Error score.** Let $$n$$ be the number of evaluation instances, $$y_i$$ the target numerical score for instance $$i$$, and $$\hat y_i$$ the model's predicted score. The regression metrics are:

$$
\begin{aligned}
\mathrm{RMSE}
&=\sqrt{\frac{1}{n}\sum_{i=1}^{n}(\hat y_i-y_i)^2},\\
\mathrm{MAE}
&=\frac{1}{n}\sum_{i=1}^{n}|\hat y_i-y_i|.
\end{aligned}
$$

The paper uses **RMSE for MANNERS-DB** and **MAE for Trust-Transfer**. For **SocialIQA**, which has discrete answer choices, the error score is instead the fraction of incorrect predictions:

$$
\mathrm{ErrorRate}
=\frac{1}{n}\sum_{i=1}^{n}\mathbf{1}[\hat c_i\neq c_i],
$$

where $$\hat c_i$$ is the predicted answer, $$c_i$$ is the correct answer, and $$\mathbf{1}[\cdot]$$ equals 1 when its condition holds and 0 otherwise. **Lower is better** for all three error metrics.

**2. Consistency with Mode (CwM).** This metric measures agreement with the majority human label. Writing $$c_i^{\mathrm{human}}$$ for the human reference class and $$\hat c_i$$ for the model's corresponding class prediction gives:

$$
\mathrm{CwM}
=\frac{1}{n}\sum_{i=1}^{n}
\mathbf{1}[\hat c_i=c_i^{\mathrm{human}}].
$$

For MANNERS-DB, human ratings of **3 or above** are treated as socially acceptable; the majority of these binarized ratings determines the reference class. Trust-Transfer uses an analogous threshold of **4 or above**, with a single participant rating per instance. SocialIQA uses its provided answer label directly. **Higher CwM is better**, with 1 indicating agreement on every instance.

**3. Distribution measures.** For a given instance, let $$q(z)$$ be the model's probability of human response $$z$$ and $$p(z)$$ the empirical fraction of annotators giving that response. The paper evaluates these measures only on **MANNERS-DB**, which has 15 ratings per instance; the other two datasets provide only one label per instance. <d-cite key="zhang2023large"></d-cite>

**Entropy relative similarity** compares the uncertainty of the two distributions. The paper defines:

$$
S(q,p)
=\frac{\sum_z q(z)\log q(z)}{\sum_z p(z)\log p(z)}
$$

- **$$S(q,p) = 1$$** indicates same entropy. 
- **$$S(q,p) < 1$$** means the model has less uncertainty than the human response distribution
- **$$S(q,p) > 1$$** means it has more uncertainty.

**2-Wasserstein distance** compares the distributions while accounting for distances between numerical rating values. For ratings $$z_j$$ and $$z_k$$, its standard discrete form is:

$$
W_2(q,p)
=\left[
\min_{\Gamma\in\Pi(q,p)}
\sum_{j,k}\Gamma_{jk}(z_j-z_k)^2
\right]^{1/2}.
$$

Here, $$\Gamma_{jk}$$ is probability mass transported from rating $$z_j$$ to rating $$z_k$$. The set $$\Pi(q,p)$$ contains nonnegative transport plans satisfying $$\sum_k\Gamma_{jk}=q(z_j)$$ and $$\sum_j\Gamma_{jk}=p(z_k)$$. Moving probability farther across the rating scale costs more. **Lower is better**, and a distance of 0 means the distributions match.

### Experiment Result

The paper's Table I below reports error scores, CwM, and distribution measures across the three datasets.

{% include figure.liquid
   path="assets/img/2026-10-21-human-models/llm-human-model-table-1-results.png"
   class="img-fluid rounded z-depth-1"
   alt="Table I compares DAVINCI, T5, and dataset-specific baselines on error scores, consistency with the human mode, entropy relative similarity, and 2-Wasserstein distance, including context-specific and overall results."
   caption="Error scores, CwM, and distribution measures."
   zoomable=true
%}

The result shows that **pretrained LLMs can capture aspects of human judgment without task-specific fine-tuning**. DAVINCI achieves aggregate prediction errors comparable to specialized baselines, and the CwM results indicate that LLM predictions can approach or exceed average human agreement with benchmark labels on MANNERS-DB and SocialIQA. However, T5's weaker performance on Trust-Transfer under the original prompts shows that predictive effectiveness varies across models and tasks.

Agreement with human labels does not necessarily imply a faithful representation of human response variability. On MANNERS-DB, DAVINCI produces lower-entropy distributions than the human ratings, whereas T5 produces higher-entropy distributions. Although T5 is closer to the empirical distributions under the Wasserstein metric, the specialized Bayesian baseline provides the closest overall match.

### LLMs on Tasks

#### From a Text Description to a Prediction

The previous section defines the input prompt as

$
x_t=f(s_t,h_t,mathcal{A}),
$

where $s_t$ is the current environment state, $h_t$ is the interaction history, and $mathcal{A}$ is the set of agents. The LLM reads $x_t$ and predicts a human action, judgment, or mental state $z_t^H$.

For a task with a discrete set of possible answers $mathcal{Z}$, the predicted answer can be written as

$
hat{z}_t^H=argmax_{zinmathcal{Z}}p_l(zmid x_t).
$

The robot first turns the situation into text. The LLM then gives a prediction about the human. The prediction can later be used by a robot planner.

#### Three Prediction Tasks

The paper evaluates this idea on three tasks. The tasks are different, so they test different parts of human modeling.

| Dataset | Human quantity to predict | Main challenge |
| --- | --- | --- |
| MANNERS-DB | Social appropriateness of a robot action, rated from 1 to 5 | Physical and spatial context |
| Trust-Transfer | Human trust after watching robot demonstrations, rated from 1 to 7 | Effect of interaction history |
| SocialIQA | The most reasonable social response among three choices | Social commonsense reasoning |

#### MANNERS-DB: Social Appropriateness

In MANNERS-DB, the model receives a text description of a robot action in a social setting. It predicts whether people will find the action appropriate.

This task is difficult because the answer depends on details in the scene. The distance between the robot and a person, the direction of motion, and the position of objects can all change the judgment. These details are easier to see in an image or a physical scene than in a short text prompt.

#### Trust-Transfer: Change in Human Trust

The Trust-Transfer task asks whether a person's trust changes after watching robot demonstrations. The input includes information about what the robot did before. The output is a trust rating.

This task is different from MANNERS-DB. Here, the main information is not only the current scene. The earlier interaction history also matters. A person may trust a robot more after several successful actions, or less after seeing a failure.

#### SocialIQA: Social Commonsense

SocialIQA contains short social situations with three possible answers. The model chooses the answer that best matches normal human behavior.

This task has less physical information than the first two tasks. It mainly tests whether the language model has learned common social knowledge from text.

#### Reading the Results

The results are given in Table I in the previous section. DAVINCI is close to specialized baselines on several tasks. T5 performs differently across the datasets.

The results also show that a correct final answer is not the whole story. A human model should also give a reasonable uncertainty distribution. On MANNERS-DB, the predicted distributions from the LLMs do not always match the spread of human ratings. This is why the paper reports both prediction error and distribution measures.

### LLMs Sensitivity to Prompt Structure

#### What Is Inside a Prompt?

The LLM does not directly see the robot scene. It only receives a text prompt. A prompt may include the current state, previous actions, task instructions, and a question about the human response.

| Prompt part | Information it gives |
| --- | --- |
| Task instruction | What the model needs to predict |
| Current state | Who is present, what objects are present, and what is happening now |
| Interaction history | What the robot and human did earlier |
| Question | The action, judgment, or trust value to predict |

#### Why Prompt Changes Matter

The paper changes the wording and structure of prompts and compares the resulting predictions. The answer can change when the order of information changes or when important context is removed.

For example, a prompt that only says “the robot moves an object” leaves out many useful details. It does not say where the object is, who is nearby, or what happened before. A longer prompt can include these details, but a longer prompt is not always better. The information still needs to be clear and relevant.

#### A Practical Limitation

This sensitivity creates a problem for real robot systems. The same physical event should not lead to very different predictions only because the text is written in a different way.

A practical system therefore needs a fixed prompt format. The format should state the task, the relevant objects, the robot action, the human action, and the needed history. It should also avoid adding unrelated details.

### Case Studies: Planning

#### Using a Human Prediction in Planning

The LLM prediction is not the final robot action. It is one input to the planner. The planner uses the predicted human response when comparing possible robot actions.

For example, the robot may need to decide whether to continue, slow down, explain its action, or choose a different object. The better choice depends partly on what the human may do next.

#### Trust-Based Table Clearing

The first case study is a simulated table-clearing task. The robot needs to clear objects from a table while considering human trust. The paper replaces a custom human model from earlier work with an LLM-based human model.

The goal is not only to remove objects quickly. The robot also needs to avoid actions that may lead to an unsuitable level of trust. A person who trusts the robot too much may fail to intervene when intervention is needed. A person with very low trust may interrupt useful robot actions.

This case study shows that an LLM-based model can be connected to a planning system. It also shows that the model is used at a high level. It does not replace motion planning or physical control.

#### Utensil-Passing Experiment

The second case study is a robot utensil-passing task with human participants. The robot passes utensils to a person. The person can respond differently depending on the utensil and the robot action.

The planner uses the predicted human response to select an action sequence. The experiment includes 65 participants. The paper compares planning with the LLM-based human model against a basic myopic plan, which only considers the immediate task outcome.

{% include figure.liquid
   path="assets/img/2026-10-21-human-models/llm-human-model-figure-5-utensil-setup.png"
   class="img-fluid rounded z-depth-1"
   alt="A robot and a human take part in the utensil-passing experiment."
   caption="Utensil-passing experiment setup used to study planning with an LLM-based human model."
   zoomable=true
%}

The LLM-based planner can make better high-level choices in this task. However, the system still needs other components. The LLM does not provide object coordinates, collision-free trajectories, or a safety guarantee.

{% include figure.liquid
   path="assets/img/2026-10-21-human-models/llm-human-model-figure-6-scissors-handover.png"
   class="img-fluid rounded z-depth-1"
   alt="A robot hands over scissors during the utensil-passing task."
   caption="Example handover case from the utensil-passing task. The predicted human response affects the robot's planning choice."
   zoomable=true
%}

### Conclusion

The paper starts by turning an interaction state and history into a text prompt. It then uses an LLM to predict a human judgment, action, or trust state.

The three dataset experiments test social appropriateness, trust, and social commonsense. These tests show that pretrained LLMs can make useful zero-shot predictions in some settings. At the same time, the prompt experiments show that the predictions are sensitive to how the situation is described.

The two planning case studies take one further step. They use the predicted human response when choosing robot actions. This is useful for high-level interaction planning.

The method still has clear limits. Language models are weak at some spatial, physical, and numerical details. A complete robot system therefore needs both an LLM-based human model and other models for perception, geometry, motion, and safety.

## Presentation Q&A:

### LESS is More: Rethinking Probabilistic Models of Human Behavior

-   The experiments show that LESS sometimes performs worse than Boltzmann. Then, does that indicate that we shouldn't consider trajectory similarity when choosing an action? In the real world, when should we treat similar choices as distinct options?
-   The authors used predefined feature vectors for computing trajectory similarity. Does this limit the performance of LESS? If so, how can we learn those features or extract them automatically?

### LLMs as Zero-Shot Human Models for Human-Robot Interaction

-   Since LLMs are only used to predict human behavior, does it matter that these models perform poorly on spatial and numerical reasoning?
-   The authors observed that LLM performance is sensitive to prompt design. Would this limitation make LLMs ineffective as human models?
-   The authors used very simple tasks to test LLMs as human models. But, in the real world, would LLMs be sufficient to approximate humans?
-   When interacting with humans, some people will do the exact opposite of what's expected, ex: intervene for a whisk but not a knife, seemingly just to be contrarian. How can an LLM or Robot account for this?

### Report Status

This report is currently being prepared and will be updated by the student reporter team following the lecture session and classroom discussion. Please check back soon!

## Appendix:

Used GPT-6.1 Sol to transfer text into a Markdown file.
