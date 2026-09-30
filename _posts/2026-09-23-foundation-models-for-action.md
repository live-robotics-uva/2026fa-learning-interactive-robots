---
layout: distill
title: "Foundation Models for Action"
description: "Vision-Language-Action (VLA) models (RT-1, RT-2, OpenVLA) trained on internet-scale multimodal data and cross-embodiment robot demonstrations."
date: 2026-09-23
future: true
htmlwidgets: true
ready: true

# Reporter team authors
authors:
  - name: "Angelica Bain, Srikar Bangaru, Samriddhi Kumar, Caroline Xu"

bibliography: 2026-09-23-foundation-models-for-action.bib

toc:
  - name: "Introduction"
  - name: "Diffusion Policy"
    subsections:
    - name: "Introduction and Problem"
    - name: "Diffusion for Robot Actions"
    - name: "Visual Conditioning and Model Architecture"
    - name: "Action Chunking and Receding-Horizon Control"
    - name: "Key Properties of Diffusion Policy"
    - name: "Experiments and Limitations"
  - name: "Vision-Language-Action Flow Model"
    subsections:
    - name: "Motivation and Design Goal"
    - name: "Model Architecture"
    - name: "Generating Actions with Conditional Flow Matching"
    - name: "Cross-Embodiment Data and Training Recipe"
    - name: "Experimental Evaluation"
    - name: "Limitations and Open Challenges"
  - name: "Cross-Paradigm Comparison"
    subsections:
    - name: "Shared Principle: Generative Action Chunks"
    - name: "Key Differences"
  - name: "Question and Answer"

---
## Introduction
Autonomous robots operating in unstructured, human-centered environments must do more than recognize objects or understand instructions, they must translate that understanding into **physical actions that succeed in the real world**. Unlike purely perceptual or language-based tasks, robot control is affected by contact dynamics, sensing uncertainty, execution error, and changes in the environment that occur as the robot acts.

A central theme of this lecture is that **generalization must survive physical execution**. Language may identify *what* task should be performed, but a robot policy must still determine *how* to move in a way that is precise, temporally consistent, and responsive to feedback.

Learning general robot policies introduces several challenges:
1. **Multiple Valid Behaviors**: A single task may admit several successful action sequences. For example, a robot may approach an object from different directions or complete subtasks in different orders. A policy therefore needs to represent distributions over possible behaviors rather than only a single deterministic action.
2. **Temporal Dependence**: Robot actions are not independent across time. Once a robot begins following a particular strategy, subsequent controls must remain consistent with that decision while still responding to changes in the environment.
3. **Physical Execution and Feedback**: Predictions must remain useful after they are executed. Contact, control error, or unexpected changes can cause the physical state to differ from what the policy anticipated, making closed-loop observation and replanning important.
4. **Generalization Across Tasks and Environments**: A general-purpose robot should ideally reuse knowledge across different tasks, objects, scenes, and robot embodiments rather than requiring a completely separate policy for every behavior.

To address these challenges, this lecture examines two approaches to learning and generating robot actions:

- **Diffusion Policy** <d-cite key="chi2023diffusion"></d-cite>: Represents robot control as a conditional generative modeling problem. Rather than directly predicting a single action, Diffusion Policy generates continuous **action sequences** through iterative denoising, allowing the policy to model multimodal behaviors while maintaining temporal consistency.

- **$\pi_0$: A Vision-Language-Action Flow Model** <d-cite key="black2025pi0"></d-cite>: Combines a pretrained vision-language model with a continuous action-generation module. Visual and language representations provide semantic task context, while a dedicated action expert generates continuous robot action chunks through flow matching.

Together, these approaches highlight two complementary components of general robot intelligence: learning expressive representations of **what behavior is appropriate** and generating continuous, closed-loop actions that can reliably realize that behavior in the physical world.


## Diffusion Policy: Visuomotor Policy Learning via Action Diffusion

### Introduction and Problem {#introduction-and-problem}

A central goal in robot learning is to learn control policies directly from demonstrations. In **behavior cloning**, demonstrations pair observations of the robot and its environment with the controls executed by a human or expert controller. The policy is then trained through supervised learning to reproduce appropriate controls from the robot's current context.

Although this resembles a standard supervised learning problem, predicting robot actions introduces several additional challenges. The Diffusion Policy paper highlights three in particular: **multimodal action distributions**, **sequential correlation between actions**, and the need for **high-precision control**.

Robot actions are sequential because a control decision at one timestep constrains which actions are sensible at the next. They must also remain precise after being physically executed: small errors can alter the state of the environment and affect all subsequent actions. Most importantly, the same situation may have several different successful solutions rather than one uniquely correct action.

**Multimodal action distributions.** A distribution is **multimodal** when it contains several distinct high-probability solutions, or modes. In robot manipulation, these modes can correspond to different strategies for accomplishing the same task. For example, a robot may be able to approach an object from either the left or the right, use different valid grasps, or complete several subtasks in different orders. Demonstrations can therefore contain substantially different actions even when the underlying task and observation are similar.

This creates a problem for simple deterministic regression. A squared-error regression objective tends to favor predictions near the average of the demonstrated actions. When the demonstrations represent distinct strategies, however, the average between them may not itself be a useful strategy. For instance, averaging trajectories that pass around opposite sides of an obstacle could result in a trajectory that passes between the demonstrated modes rather than following either successful path.

The objective is therefore not always to recover one "correct" action. A useful robot policy may instead need to represent a **distribution over several possible successful behaviors**.

**Representing an action distribution.** The paper and presentation compare three broad ways to represent this distribution:

| Policy Representation | Basic Idea | Main Trade-off |
| :--- | :--- | :--- |
| **Explicit / Direct Prediction** | Directly predict an action or the parameters of an action distribution | Simple and efficient, but common regression objectives can struggle when distinct behaviors are all valid |
| **Implicit / Energy-Based Policy** | Learn an energy function over candidate actions and search for actions with low energy | Can represent multimodal distributions, but action optimization and training can be difficult |
| **Diffusion Policy** | Begin with a noisy action sample and iteratively refine it through learned updates | Can represent complex, high-dimensional action distributions, but requires multiple denoising steps during inference |

{% include figure.liquid
   path="assets/img/2026-09-23-foundation-models-for-action/diffusion-policy-representations.png"
   class="img-fluid rounded z-depth-1"
   caption="Figure 1: Three approaches to representing robot action distributions. An explicit policy directly predicts an action or distribution parameters; an implicit policy learns an energy function over candidate actions; Diffusion Policy begins from noise and iteratively refines the sample using a learned gradient field. Adapted from Chi et al."
%}

Diffusion Policy <d-cite key="chi2023diffusion"></d-cite> takes the third approach. Instead of treating robot behavior as a single deterministic mapping, it represents a **distribution over possible action sequences** and generates one trajectory through a conditional denoising process. This changes the policy-learning problem from **direct action prediction** to **generative action modeling**. Because the model generates from a distribution, it can represent several distinct behaviors without forcing them into a single averaged prediction.

Diffusion Policy also generates **sequences of actions rather than isolated controls**. This is important because robot actions are temporally dependent: once the robot begins following one strategy, subsequent actions should remain consistent with that decision. Jointly generating a sequence gives the policy a way to model these dependencies and produce coherent trajectories over multiple control steps.

### Diffusion for Robot Actions {#diffusion-for-robot-actions}

Diffusion Policy adapts **denoising diffusion models** to robot control. In a conventional diffusion model, the goal is to learn how to reverse a process that gradually corrupts data with noise. Diffusion Policy applies the same idea to robot behavior: instead of denoising an image, the model denoises an **action sequence**.

During training, the model begins with a clean action chunk from a robot demonstration and adds Gaussian noise at a randomly selected diffusion level. The presentation describes this corruption process as

$$
A^k = \sqrt{\bar{\alpha}_k} A^0
      + \sqrt{1-\bar{\alpha}_k}\epsilon.
$$

Where:

- $$A^0$$ is the original, clean action chunk from the demonstration
- $$A^k$$ is the corrupted version of that action chunk at diffusion step $$k$$
- $$k$$ is the **diffusion timestep**, indicating the current noise level
- $$\bar{\alpha}_k$$ controls how much of the original action signal is retained at diffusion step $$k$$
- $$\epsilon$$ is sampled Gaussian noise
- $$\sqrt{\bar{\alpha}_k}A^0$$ represents the portion of the demonstrated action sequence that remains
- $$\sqrt{1-\bar{\alpha}_k}\epsilon$$ represents the noise added to the action sequence

The network is then trained to predict the noise that was injected into the demonstrated action chunk. The training objective is:

$$
\mathcal{L}
=
\mathbb{E}
\left[
\left\|
\epsilon -
\epsilon_\theta(A^k,o,k)
\right\|^2
\right].
$$

Where:

- $$\mathcal{L}$$ is the loss minimized during training
- $$\mathbb{E}$$ indicates that the loss is averaged across demonstrated action chunks, sampled noise, and different diffusion levels
- $$\epsilon$$ is the actual Gaussian noise that was added
- $$\epsilon_\theta$$ is the neural network trained to predict that noise
- $$\theta$$ denotes the learned parameters of the network
- $$A^k$$ is the corrupted action chunk
- $$o$$ is the observation that provides the robot's current context
- $$k$$ tells the network the current diffusion noise level
- $$\|\cdot\|^2$$ measures the squared error between the true injected noise and the model's prediction

The training problem can therefore be interpreted as **learning to reverse corruption**. Given a noisy action sequence and the robot's current observation, the network learns which part of the sequence should be treated as noise.

At inference time, this process is reversed. There is no demonstrated action chunk to corrupt. Instead, the policy:

1. encodes the current robot observation,
2. initializes a random Gaussian sample with the dimensions of the full action chunk,
3. repeatedly predicts and removes noise over $$K$$ denoising steps, conditioning each step on the observation, and
4. returns the final denoised sequence as a candidate robot action chunk

Here, $$K$$ denotes the total number of iterative denoising steps used to transform the initially random sample into a structured action sequence.

An important consequence of this generative formulation is that **different random initializations can produce different valid behaviors for the same observation**. Rather than collapsing several demonstrated strategies into one average prediction, the diffusion process can generate samples corresponding to different modes of the learned action distribution. <d-cite key="chi2023diffusion"></d-cite>

### Visual Conditioning and Model Architecture {#visual-conditioning-and-model-architecture}

The denoising process described above cannot generate useful robot actions from the noisy action chunk alone. The generated trajectory must also depend on **what the robot currently observes**.

Diffusion Policy therefore models a conditional action distribution

$$
p(A_t \mid O_t),
$$

where:

- $$A_t$$ is the sequence of future actions being generated beginning at robot time $$t$$.
- $$O_t$$ represents the robot's recent observations.
- $$p(A_t \mid O_t)$$ is the probability distribution over possible action sequences conditioned on those observations.

Rather than jointly generating future observations and actions, Diffusion Policy directly generates actions **conditioned on the current observation**. This avoids the additional computational cost of predicting future visual states and makes the formulation more suitable for real-time robot control. 

Visual observations are first passed through a visual encoder to produce a compact representation that can be reused throughout the denoising process. The paper uses a modified **ResNet-18** visual encoder and trains it jointly with the policy. Different camera views are encoded separately and then combined into the observation representation. <d-cite key="chi2023diffusion"></d-cite>

The paper considers two architectures for the noise-prediction network $$\epsilon_\theta$$: a **temporal CNN** and a **Transformer**.

{% include figure.liquid
   path="assets/img/2026-09-23-foundation-models-for-action/diffusion-policy-overview.png"
   class="img-fluid rounded z-depth-1"
   caption="Figure 2: Diffusion Policy overview. The policy conditions repeated denoising of an action sequence on recent robot observations. The paper implements the noise-prediction network using either a temporal CNN with FiLM conditioning or a Transformer with cross-attention. Adapted from Chi et al."
%}

**CNN-based Diffusion Policy.** The CNN implementation applies one-dimensional temporal convolutions along the action sequence. This allows the model to capture local temporal relationships between neighboring actions.

The observation representation is incorporated using **Feature-wise Linear Modulation (FiLM)**. As illustrated in Figure 2, the observation-derived conditioning parameters modify intermediate CNN features during denoising. The diffusion timestep $$k$$ is also provided to the network so that it knows the current noise level.

The CNN-based implementation performed well across many of the evaluated tasks and generally required relatively little task-specific hyperparameter tuning. However, the paper notes that temporal convolution can over-smooth signals when the desired actions change rapidly over time. 

**Transformer-based Diffusion Policy.** The Transformer implementation instead represents the noisy action sequence as a sequence of **action embeddings**. The robot observation is converted into a separate sequence of **observation embeddings**.

Within each Transformer decoder block, **cross-attention** allows the action representations to access information from the observation. In this way, the network can use the current scene and robot state to determine how each component of the noisy action trajectory should be updated. The Transformer also uses causal attention over the action sequence, so each action representation attends only to itself and previous action representations. This preserves the temporal structure of the predicted trajectory.

The Transformer architecture is particularly useful for tasks involving more complex or rapidly changing action sequences. In the paper's state-based experiments, Transformer variants often performed best when task complexity and the rate of action change were high. However, the authors also found the Transformer to be more sensitive to hyperparameter choices than the CNN implementation. <d-cite key="chi2023diffusion"></d-cite>

Both architectures ultimately serve the same role: given the current noisy action sequence, the robot observation, and the diffusion timestep, they predict the noise that should be removed. Repeating this prediction over multiple denoising steps transforms an initially random action chunk into one that is consistent with the robot's current physical context.

### Action Chunking and Receding-Horizon Control {#action-chunking-and-receding-horizon-control}

Diffusion Policy generates **sequences of future actions**, or action chunks, rather than predicting each control independently. In the notation used during the lecture,

$$
A_t = [a_t, \ldots, a_{t+H-1}],
$$

where $$A_t$$ is the action chunk generated at robot time $$t$$, $$a_t$$ is an individual control command, and $$H$$ is the number of future actions in the sequence.

Predicting several actions jointly helps capture **temporal dependence**: once the robot begins following one strategy, later actions should remain consistent with that decision. However, the robot does not execute the entire chunk. Instead, it executes only the first $$h$$ actions, where $$ h \leq H, $$

then receives a new observation and generates another action chunk. This is a form of **receding-horizon control**, in which the policy plans farther ahead than it commits.

A question raised during the lecture was: **Why is $$h \leq H$$? If the robot only executes $$h$$ actions, why generate a longer sequence of $$H$$ actions?**

The two horizons serve different purposes. A longer prediction gives the policy enough temporal context to generate a coherent trajectory, while executing only a shorter prefix allows the robot to re-observe and correct for contact dynamics, sensing noise, or execution error before committing too far into the future.

In the paper's notation, this is described using three horizons: <d-cite key="chi2023diffusion"></d-cite>

- **Observation horizon ($$T_o$$):** number of recent observations given to the policy.
- **Prediction horizon ($$T_p$$):** number of future actions predicted.
- **Action horizon ($$T_a$$):** number of predicted actions executed before replanning.

Thus, the lecture's $$H$$ corresponds conceptually to the prediction horizon, while $$h$$ corresponds to the shorter execution horizon. This creates a trade-off between **temporal consistency and responsiveness**.

{% include figure.liquid
   path="assets/img/2026-09-23-foundation-models-for-action/diffusion-policy-horizon-ablation.png"
   class="img-fluid rounded z-depth-1 mx-auto d-block"
   width="75%"
   max-width="75%"
   caption="Figure 3: Action-horizon and latency ablations for Diffusion Policy. Increasing the action horizon initially improves temporal consistency, but very long execution horizons reduce responsiveness to new observations. The latency experiment evaluates how performance changes when delays are introduced between observation and action execution. Adapted from Chi et al."
%}

The action-horizon ablation demonstrates this trade-off experimentally. Performance initially improves as the action horizon increases, but eventually decreases when the robot remains open-loop for too long. The paper reports that an action horizon of eight steps performed best for most evaluated tasks. <d-cite key="chi2023diffusion"></d-cite>

### Key Properties of Diffusion Policy {#key-properties-of-diffusion-policy}


**Multimodal behavior and temporal consistency.** One of Diffusion Policy's main advantages is its ability to represent multiple valid behaviors without averaging them together. Because inference begins from a random action sample, different rollouts can converge to different modes of the learned action distribution.

The Push-T example illustrates this clearly: from the same state, the robot can move around either the left or right side of the T-shaped block before pushing it toward the target.

{% include figure.liquid
   path="assets/img/2026-09-23-foundation-models-for-action/diffusion-policy-multimodal-behavior.png"
   class="img-fluid rounded z-depth-1 mx-auto d-block"
   width="80%"
   max-width="80%"
   caption="Figure 4: Multimodal behavior in the Push-T task. Diffusion Policy represents both the left and right approaches while committing to a single coherent mode within each rollout. Adapted from Chi et al."
%}

Diffusion Policy also maintains **temporal consistency** by generating an entire action sequence jointly. This prevents consecutive actions from switching between different valid strategies. The paper demonstrates both **short-horizon multimodality**, such as approaching an object from different directions, and **long-horizon multimodality**, where subtasks can be completed in different valid orders.

**Synergy with position control.** Another important design decision is the representation of the robot's actions. Many previous behavior-cloning systems use **velocity control**, where the policy predicts how fast and in what direction the robot should move. Diffusion Policy instead performs particularly well with **position control**, where the policy predicts desired positions directly.

{% include figure.liquid
   path="assets/img/2026-09-23-foundation-models-for-action/diffusion-policy-position-control.png"
   class="img-fluid rounded z-depth-1 mx-auto d-block"
   width="70%"
   max-width="70%"
   caption="Figure 5: Effect of switching from velocity to position control. While the evaluated baseline methods generally lose performance under position control, Diffusion Policy is able to benefit from the position-based action representation. Adapted from Chi et al."
%}

The authors suggest two reasons for this result. First, position-control actions can produce more pronounced multimodality because there may be several different desired positions that lead to successful behavior. Second, position control is less affected by **compounding error** when predicting sequences of future actions. With velocity control, a small error in one predicted velocity changes the resulting position and can affect every later command. Position targets provide a more direct reference for where the robot should move. <d-cite key="chi2023diffusion"></d-cite>

**High-dimensional action-sequence prediction.**  Diffusion models scale well to high-dimensional outputs, allowing Diffusion Policy to predict full action sequences rather than isolated controls. This supports the temporal consistency discussed above and reduces the need to make each decision independently.

Sequence prediction also provides robustness to **idle actions** in demonstrations. During teleoperation, a demonstrator may pause temporarily, creating repeated position commands or near-zero velocities. Single-step behavior-cloning policies can overfit to these pauses and become stuck. By modeling actions as part of a longer sequence, Diffusion Policy can better represent the broader motion surrounding an idle period rather than treating the pause as an isolated target action. 

**Training stability.** Finally, the paper reports that Diffusion Policy is relatively stable to train compared with implicit energy-based policies such as IBC. Energy-based policies often require negative sampling during training and can exhibit unstable evaluation performance even while their training objective decreases. Diffusion Policy instead learns the gradient used to denoise actions without requiring the same normalization procedure, resulting in more consistent training behavior and less task-specific hyperparameter tuning. <d-cite key="chi2023diffusion"></d-cite>

Together, these properties explain why the diffusion representation is useful beyond simply being another way to predict actions. It provides a policy representation that can capture multiple possible behaviors, maintain consistency across time, scale to action sequences, and work effectively with position-based robot control.

### Experiments and Limitations {#experiments-and-limitations}
Diffusion Policy was evaluated across **15 tasks from four robot-manipulation benchmarks**, spanning both simulated and real-world environments. The paper reports an average **46.9% improvement in success rate** over the compared behavior-cloning methods. <d-cite key="chi2023diffusion"></d-cite>

The evaluation includes tasks with different action dimensions, state- and image-based observations, single- and multi-stage manipulation, and both rigid and fluid objects. These results suggest that the benefits of Diffusion Policy are not limited to a single task or setting. 

Despite these results, the method has two important limitations.

**Dependence on demonstration data.** Because Diffusion Policy is trained through behavior cloning, its performance still depends on the quality and coverage of the demonstrations. Inadequate demonstration data can lead to poor performance even if the action distribution is modeled effectively.

**Inference latency.** Diffusion Policy requires **multiple denoising steps during inference**, making it more computationally expensive than policies that generate actions in a single forward pass. The authors use DDIM to reduce the number of inference iterations, but the paper notes that the remaining computational cost may still be too high for tasks requiring very high-rate control. 

Overall, the experiments provide broad evidence for Diffusion Policy across a range of manipulation tasks, while its reliance on demonstration data and iterative inference remain important limitations for real-time deployment.

**Additional resource.** The presenting team also recommended [*Diffusion Policy: LeRobot Research Presentation #2 by Cheng Chi*](https://www.youtube.com/watch?v=M03sZFfW-qU) for a more detailed walkthrough of the method and its design choices.

## $\pi_0$: A Vision-Language-Action Flow Model for General Robot Control

### Motivation and Design Goal {#pi0-motivation-and-design-goal}

$\pi_0$ extends generative action-sequence modeling into a **generalist vision-language-action (VLA) model**. Diffusion Policy shows that a policy can model a distribution over continuous action chunks, but it is normally trained for a particular task family from robot demonstrations. $\pi_0$ asks whether continuous generative control can instead be combined with the broad semantic knowledge of a pretrained vision-language model and the physical experience contained in a large, cross-embodiment robot dataset. <d-cite key="black2025pi0"></d-cite>

The paper identifies three obstacles to building robot foundation models. First, the research must operate at a sufficiently large scale for pretraining to provide meaningful transfer. Second, the architecture must combine heterogeneous inputs and robot embodiments while preserving the precision required for manipulation. Third, the training recipe must determine how broad, diverse pretraining data and smaller, higher-quality post-training data should be combined. $\pi_0$ addresses these obstacles with a pretrained VLM backbone, a robotics-specific **action expert**, conditional flow matching, and a pretraining/post-training procedure modeled after the recipe used for language models. <d-cite key="black2025pi0"></d-cite>

The resulting policy models

$$
p(A_t \mid o_t)
$$

where $A_t=[a_t,\ldots,a_{t+H-1}]$ is a chunk of future actions and $o_t=[I_t^1,\ldots,I_t^n,\ell_t,q_t]$ contains multiple camera images, the language instruction, and the robot's proprioceptive state. In the experiments, the action horizon is **$H=50$**. This formulation connects the semantic question of *what task should be performed* with the control question of *what continuous motion should be executed next*?

### Model Architecture {#pi0-model-architecture}

{% include figure.liquid
   path="assets/img/2026-09-23-foundation-models-for-action/pi0_framework.png"
   class="img-fluid rounded z-depth-1"
   caption="Figure 6: Overview of the $\pi_0$ framework. A broad pretraining mixture is used to train a VLA model composed of a pretrained PaliGemma backbone and a smaller action expert. The resulting policy can control multiple robot embodiments and can be post-trained for demanding downstream tasks. Adapted from Black et al."
%}

The main network begins with **PaliGemma**, an open-source 3-billion-parameter VLM. Image encoders convert two or three camera observations into embeddings, while the language command is represented as language tokens. The robot's joint state is projected into the same embedding space. These inputs provide the context that the model uses to generate its next action chunk. <d-cite key="black2025pi0"></d-cite>

$\pi_0$ augments the VLM with a separate **300-million-parameter action expert**, resulting in a model with about **3.3 billion parameters**. The VLM weights process image and language inputs, while a separate set of weights processes the robot state and continuous action tokens. The paper describes this structure as similar to a two-component mixture of experts: One component preserves the VLM's semantic representations, while the other specializes in robot-specific inputs and outputs. All action tokens can attend bidirectionally to one another, which lets the model coordinate the full action chunk rather than predicting each control independently. <d-cite key="black2025pi0"></d-cite>

This separation is important because language tokens and robot actions have different mathematical structures. Language is discrete and is typically trained with next-token cross-entropy, while robot actions are continuous, high-frequency values that require fine precision. Instead of forcing continuous controls into the VLM's discrete vocabulary, $\pi_0$ retains the semantic backbone but trains the action expert with a continuous flow-matching objective.

### Generating Actions with Conditional Flow Matching {#pi0-conditional-flow-matching}

Given the current observation $o_t$ and a demonstrated action chunk $A_t$, training samples Gaussian noise $\epsilon\sim\mathcal{N}(0,I)$ and a flow timestep $\tau\in[0,1]$. It constructs an intermediate action sample along a linear path between noise and the demonstrated action:

$$
A_t^\tau = \tau{A}_t + (1-\tau)\epsilon.
$$

When $\tau=0$, the sample is pure noise; when $\tau=1$, it equals the demonstrated action chunk. The action expert learns the **transport velocity** that moves the intermediate sample toward the demonstrated actions. For this path, the target vector field is

$$
u(A_t^\tau \mid A_t)=A_t-\epsilon.
$$

The conditional flow-matching loss is

$$
L^\tau(\theta)=
E\left[
\left\|
v_\theta(A_t^\tau,o_t)
-
u(A_t^\tau\mid{A}_t)
\right\|^2
\right].
$$

Here, $v_\theta$ is the velocity predicted by the action expert and $u$ is the target direction derived from the demonstration and sampled noise. Minimizing their squared difference teaches the model how a noisy candidate chunk should move at each flow timestep. It is important to note that, **$\tau$ is generation time, not physical robot time $t$**. <d-cite key="black2025pi0"></d-cite>

At inference time, the policy starts with random noise $\mathbf{A}_t^0$ and repeatedly follows the learned vector field using forward Euler integration:

$$
A_t^{\tau+\delta}
=
A_t^\tau
+
\delta{v}_\theta(A_t^\tau,o_t).
$$

The paper uses **10 integration steps**, with $\delta=0.1$. Each step asks the model how every component of the candidate action chunk should change, multiplies that velocity by the step size, and adds the update to the current chunk. At $\tau=1$, the model returns a complete 50-action sequence. The robot executes only a prefix before generating another chunk from a new observation. The reported systems execute 16 actions at 20 Hz or 25 actions at 50 Hz before inference is run again. <d-cite key="black2025pi0"></d-cite>

Flow matching and diffusion therefore share the same broad generative idea: begin from noise and iteratively produce a coherent action sequence. The training targets differ, however. Diffusion Policy learns the noise that should be removed at each denoising step, while $\pi_0$ directly learns a velocity field along a path from noise to demonstrated actions. The presentation emphasized that both approaches can represent multimodal continuous behaviors, but flow matching gives $\pi_0$ a direct way to integrate continuous action generation with a pretrained VLM.

### Cross-Embodiment Data and Training Recipe {#pi0-data-and-training}

Architecture alone is not enough to make $\pi_0$ a foundation model. The authors pretrain on roughly **10,000 hours of robot data**, including their own dexterous manipulation datasets and public datasets such as OXE. Their internal data covers **7 robot configurations and 68 tasks**, while OXE contributes data collected from **22 robots**. The embodiments include single-arm, bimanual, and mobile manipulators with different camera setups, control frequencies, kinematics, and action dimensions. <d-cite key="black2025pi0"></d-cite>

The paper separates training into two stages:

1. **Pretraining for breadth.** The model is exposed to diverse tasks, scenes, embodiments, behaviors, mistakes, and recovery trajectories. The goal is a broadly capable base policy that can follow language commands and perform many tasks at a basic level.
2. **Post-training for fluency.** The base model is fine-tuned on curated, task-specific data. This stage teaches consistent and efficient strategies for demanding downstream tasks such as laundry folding, table bussing, and box assembly.

This division provides a useful interpretation of data quality. Training only on polished demonstrations may produce a brittle policy because it rarely observes mistakes or recoveries. Training only on broad, lower-quality data may provide coverage but fail to produce efficient execution. The combination allows pretraining to supply behavioral diversity and recovery experience while post-training emphasizes the desired strategy. <d-cite key="black2025pi0"></d-cite>

Language supervision also appears at multiple levels. Short trajectory segments receive fine-grained language labels, and a high-level VLM policy can decompose a longer task into intermediate natural-language commands. The low-level $\pi_0$ policy then turns each command into continuous action chunks. This division is useful for temporally extended tasks because high-level semantic planning and high-frequency physical control operate at different timescales.

### Experimental Evaluation {#pi0-experimental-evaluation}

The experiments evaluate four main questions: whether the pretrained base model can perform tasks directly, whether VLM initialization improves language following, whether pretraining helps the model learn new dexterous tasks, and whether post-training can produce complex multi-stage behavior. The evaluation includes shirt folding, table bussing, grocery bagging, removing toast from a toaster, towel folding, placing containers in a microwave, replacing a paper-towel roll, laundry handling, box assembly, egg packing, and other manipulation tasks. <d-cite key="black2025pi0"></d-cite>


{% include figure.liquid
   path="assets/img/2026-09-23-foundation-models-for-action/pi0-out-of-box-results.png"
   class="img-fluid rounded z-depth-1"
   caption="Figure 7: Out-of-box evaluation after pretraining. The full pi0 model and its compute-parity version outperform the reported OpenVLA, Octo, and pi0-small baselines across the five evaluated tasks. Adapted from Black et al."
%}

For the **out-of-box evaluation**, the same pretrained policy is prompted to perform five tasks without task-specific post-training. The full model performs best across all tasks, and the version trained for the same number of update steps as the baselines still outperforms OpenVLA, Octo, and $\pi_0$-small. The presentation appropriately added an important caveat: these comparisons do not isolate a single cause. OpenVLA uses autoregressive action tokens, Octo uses diffusion-based chunks, and $\pi_0$ uses flow matching, but the models also differ in scale, initialization, capacity, and training. Therefore, the result supports the complete $\pi_0$ design, not a clean claim that flow matching alone causes the improvement. <d-cite key="black2025pi0"></d-cite>

The comparison with **$\pi_0$-small** similarly suggests that VLM initialization is beneficial, but it is not a controlled VLM-only ablation. The full model has 3.3B parameters and pretrained PaliGemma weights, whereas $\pi_0$-small has 470M parameters and no VLM initialization. Since both model size and initialization change, the experiment cannot fully separate their contributions. This nuance is important when interpreting the reported improvement in semantic instruction following.

The fine-tuning experiments provide stronger evidence that broad robot pretraining can improve data efficiency. Across downstream tasks, the pretrained model frequently outperforms the same architecture trained from scratch, sometimes by as much as **2x**, with larger gains on tasks that resemble behaviors seen during pretraining. The benefit is not uniform, however: transfer depends on task difficulty and similarity to the pretraining distribution. <d-cite key="black2025pi0"></d-cite>

{% include figure.liquid
   path="assets/img/2026-09-23-foundation-models-for-action/pi0_results.png"
   class="img-fluid rounded z-depth-1 mx-auto d-block"
   width="75%"
   max-width="75%"
   caption="Figure 8: Post-training results on complex, multi-stage tasks. A score of 1.0 represents complete execution, while fractional scores represent partial progress. The model initialized from broad pretraining and then post-trained performs best overall and exceeds 50% of the maximum score on every reported task. Adapted from Black et al."
%}

For the most difficult tasks, the paper compares the complete pretraining-plus-post-training procedure with two ablations: using the pretrained policy directly and training only on the task-specific data from scratch. As shown in Figure 8, the full procedure performs best overall and achieves more than 50% of the maximum score on every reported task. The largest benefits often occur on the hardest tasks, supporting the argument that broad pretraining supplies reusable behaviors and recovery strategies that curated task data alone may not contain. <d-cite key="black2025pi0"></d-cite>

{% include figure.liquid
   path="assets/img/2026-09-23-foundation-models-for-action/pi0-complex-task-montage.png"
   class="img-fluid rounded z-depth-1"
  width="75%"
   max-width="75%"
   caption="Figure 9: Examples of the complex multi-stage tasks used for post-training evaluation, including laundry folding, table bussing, box assembly, egg packing, and to-go-box packing. Adapted from Black et al."
%}

The qualitative task montage from the paper would also be valuable here because a normalized score does not fully communicate the physical complexity of the experiments. Folding deformable clothing, bracing cardboard with two arms, grasping fragile eggs, and sorting previously unseen table objects require different forms of dexterity and failure recovery. The presentation's box-building demonstration made this point especially clear: the policy coordinates both arms, uses the table as support, and retries folds when the material does not behave as expected.

### Limitations and Open Challenges {#pi0-limitations}

**Unseen embodiments.** Although $\pi_0$ is trained across several robot embodiments and a single policy can control robots with different action spaces, the paper does not perform a clean zero-shot embodiment test in which an entire robot is excluded from pretraining and then evaluated without adaptation. Cross-embodiment training is therefore evidence of shared learning across known embodiments, not yet proof of universal transfer to a completely unseen robot. <d-cite key="black2025pi0"></d-cite>

**Confounded architectural comparisons.** Several experiments change more than one factor at a time. The $\pi_0$ versus $\pi_0$-small comparison changes both VLM initialization and model size, while the OpenVLA and Octo comparisons also change the action decoder, capacity, and training setup. These results establish that the complete $\pi_0$ system is strong, but they do not cleanly determine how much of the improvement comes from flow matching, scale, semantic pretraining, action chunking, or data. <d-cite key="black2025pi0"></d-cite>

**Open-loop chunk execution.** Action chunks improve temporal consistency and amortize the cost of inference, but executing a longer prefix delays the next opportunity to correct mistakes. The paper reports that the evaluated robots execute 16 or 25 actions before replanning. Unexpected contact, a human intervention, or an object slipping can therefore occur during an open-loop interval. A useful extension would study when a policy should interrupt its current chunk and request a new observation or human assistance. <d-cite key="black2025pi0"></d-cite>

**Data composition and reliability.** The authors combine all available pretraining sources, but they do not establish the optimal proportion of tasks, embodiments, successes, failures, or recovery behaviors. Performance also remains below perfect on several tasks, and it is difficult to predict how much additional data is required for a new behavior. The paper identifies broader transfer to domains such as navigation, autonomous driving, and legged locomotion as an open question. <d-cite key="black2025pi0"></d-cite>

Finally, the model is not evaluated for **in-context robot learning**, where a new skill would be specified through examples in the prompt without parameter updates. The reported adaptation mechanism is fine-tuning, so determining whether a VLA can acquire new physical behavior through context alone remains a useful direction for future work.

## Cross-Paradigm Comparison

### Shared Principle: Generative Action Chunks {#shared-generative-action-chunks}

Diffusion Policy and $\pi_0$ operate at different scales, but they share an important design principle: both represent robot behavior as a **conditional distribution over action sequences** rather than as a single independently predicted control. This supports multimodal behavior because several distinct trajectories can be valid, and it supports temporal consistency because an entire chunk is generated jointly. Both approaches also use receding-horizon execution: generate a chunk, execute only part of it, observe the world again, and then produce another chunk. <d-cite key="chi2023diffusion"></d-cite> <d-cite key="black2025pi0"></d-cite>

Their generative procedures are closely related but not identical. Diffusion Policy begins with noise and repeatedly predicts the noise to remove, whereas $\pi_0$ learns a transport velocity and integrates that vector field from noise toward the action distribution. In both cases, iterative generation is more computationally expensive than direct regression, but it provides an expressive representation for high-dimensional and multimodal continuous actions.

### Key Differences {#cross-paradigm-key-differences}

| Dimension | Diffusion Policy | $\pi_0$ |
| :--- | :--- | :--- |
| **Primary goal** | Learn a strong visuomotor behavior-cloning policy for a task or task family | Build a generalist VLA policy that transfers across tasks and robot embodiments |
| **Training scale** | Task-specific demonstration datasets across 15 evaluated manipulation tasks | Roughly 10,000 hours of robot data, 7 internal robot configurations, 68 internal tasks, and public cross-embodiment data |
| **Conditioning inputs** | Current visual or state observation | Multiple images, language instruction, and proprioceptive robot state |
| **Semantic initialization** | No pretrained VLM is required | PaliGemma supplies Internet-scale visual-language representations |
| **Action generator** | Temporal CNN or transformer denoising network | 300M-parameter action expert attached to a 3B-parameter VLM backbone |
| **Training target** | Predict the Gaussian noise added to a demonstrated action chunk | Predict the velocity field $\mathbf{A}_t-\epsilon$ along a path from noise to the demonstrated chunk |
| **Inference** | Iterative denoising, commonly accelerated with DDIM | Ten forward-Euler flow-integration steps |
| **Action horizon** | Tuned per task; only a prefix is executed before replanning | Generates 50 actions and executes an embodiment-dependent prefix |
| **Main strength** | Precise, multimodal, temporally consistent control from demonstrations | Combines semantic instruction following, cross-embodiment pretraining, and continuous dexterous control |
| **Main limitation** | Iterative inference and dependence on task-relevant demonstrations | Large data and compute requirements, confounded ablations, and limited evidence for truly unseen embodiments |

The most important difference is therefore not simply **diffusion versus flow matching**. Diffusion Policy primarily contributes a continuous generative policy representation, while $\pi_0$ embeds a related continuous generator inside a much larger foundation-model training framework. $\pi_0$ adds semantic initialization, language conditioning, cross-embodiment data, model scale, and a pretraining/post-training recipe. Comparing the two as if only their loss functions differed would miss most of what distinguishes them.

## Presentation Q&A

During the lecture, several technical questions were raised by the presenting team concerning the capabilities and limitations of Vision-Language-Action (VLA) models. Not every question could be fully discussed during the presentation, so the summaries below reflect the points that were actually raised in class.

**Question 1:** Do you expect VLAs (Vision-Language-Action models) to exhibit the same scaling properties, or neural scaling laws, as LLMs? Why or why not?

The discussion emphasized that scaling VLAs is more difficult than scaling language models because the available data is fundamentally different. LLMs can be trained on extremely large quantities of text collected from the internet, while high-quality robot action data requires interaction with the physical world and is much more expensive to collect. Robot demonstrations also depend on the robot embodiment, sensors, action space, and physical environment, making the data less interchangeable than text.

This suggests that simply increasing model size may not produce the same improvements seen in LLMs unless robot datasets also increase substantially in scale and diversity. The class discussion therefore focused on **data availability as a major bottleneck for VLA scaling**, particularly because physical interaction data is much harder to obtain than internet-scale language or image data.

**Question 2:** Do you expect VLAs to be a sufficient approach to master all physical intelligence tasks?

The discussion was more skeptical that the current VLA formulation alone would be sufficient for all physical intelligence. A recurring point in the lecture was that **understanding what task should be performed is not the same as successfully executing it**. Vision-language models provide useful semantic knowledge, but robot policies must also generate precise and temporally consistent continuous actions that remain successful under contact, execution error, and changes in the environment.

The discussion also raised the limitation that many VLA architectures are still largely built around models originally designed for language. While this provides strong semantic reasoning, it does not necessarily mean that the architecture is naturally suited for representing continuous robot motion. The need to fine-tune policies for individual tasks or embodiments was also discussed as a potential barrier to truly general-purpose physical intelligence.

Thus, the discussion suggested that VLAs are a promising direction for combining semantic understanding with control, but broader physical intelligence may still require better action representations, stronger closed-loop feedback, and architectures designed more directly around interaction with the physical world.

**Question 3:** Do you think backpropagating through the LLM during VLA training could improve the learned representations of the LLM?

This question was raised during the presentation but was **not discussed in enough depth to reach a clear conclusion**. One related issue discussed during the lecture was whether keeping a pretrained language or vision-language backbone largely fixed limits how much the representation can adapt to robot-specific information. However, the class did not reach a detailed conclusion about whether backpropagating through the full LLM would improve VLA representations or whether the additional training cost and adaptation would be worthwhile.
