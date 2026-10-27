---
layout: distill
title: "Intent/Trajectory Prediction"
description: "Forecasting human motion and navigation trajectories in shared physical environments using generative and sequence models."
date: 2026-10-26
future: true
htmlwidgets: true
ready: true

# Reporter team authors
authors:
  - name: "Vu-Anh Le"
    affiliations:
      - name: "University of Virginia"
  - name: "Jameson Sandler"
    affiliations:
      - name: "University of Virginia"
  - name: "Anqi Li"
    affiliations:
      - name: "University of Virginia"
  - name: "Sudeesh Murugesan"
    affiliations:
      - name: "University of Virginia"

bibliography: 2026-10-26-intent-trajectory-prediction.bib

toc:
  - name: "Introduction"
  - name: "Social GAN: Socially Acceptable Trajectories with Generative Adversarial Networks"
  - name: "Estimating Cognitive Biases with Attention-Aware Inverse Planning"
  - name: "Comparison and Discussion"
  - name: "References"
---

{% include figure.liquid
   path="assets/img/2026-10-26-intent-trajectory-prediction/dynamic_intent_queries.png"
   class="img-fluid rounded z-depth-1"
   alt="Static intention points in green and road-aligned dynamic intention points in blue, with the agent and its trajectory in red."
   caption='Comparison of static (green) and dynamic (blue) intention points, with the agent and its trajectory shown in red. Reproduced from Demmler et al. (2025), Figure 1, in <em>Dynamic Intent Queries for Motion Transformer-based Trajectory Prediction</em> <d-cite key="demmler2025dynamic"></d-cite>.'
%}

## Introduction

Predicting human intent and future trajectories is an important problem in human-robot interaction. When robots work in environments with people, they need to consider not only their own actions but also how people around them may behave. For example, a delivery robot moving through a crowded area needs to predict pedestrian movements to avoid collisions and navigate safely. However, human behavior is difficult to predict because people can have different goals, interact with each other, and make decisions based on incomplete information.

This report discusses two papers that study human behavior prediction from different perspectives. The first paper, Social GAN: Socially Acceptable Trajectories with Generative Adversarial Networks, focuses on predicting multiple possible pedestrian trajectories by modeling social interactions. The second paper, Estimating Cognitive Biases with Attention-Aware Inverse Planning, considers how limited attention and cognitive biases affect human decisions. Instead of assuming that people always make optimal decisions based on complete information, this approach tries to infer what people may attend to or ignore from their observed behavior.

Although the two papers use different methods, both address important challenges in predicting human behavior. Social GAN focuses on generating diverse and socially acceptable future trajectories, while attention-aware inverse planning provides a way to explain behavior through underlying attentional biases. In this report, we review their main approaches, experimental results, and limitations, and discuss how these ideas may help robots interact more safely with humans.

## Social GAN: Socially Acceptable Trajectories with Generative Adversarial Networks

### Motivation and Problem

Predicting pedestrian movement in crowded environments is challenging because people do not move independently. Their future paths depend on both their previous movements and the behavior of others around them. For example, when two pedestrians approach each other, they may change direction, slow down, or choose different paths to avoid a collision. These different behaviors can all be reasonable, which means that predicting only one future trajectory may not be sufficient.

Previous methods have used hand-crafted features or recurrent neural networks to model pedestrian movements. However, many of these approaches mainly consider interactions within a local neighborhood. Another limitation is that minimizing the L2 prediction error may encourage the model to learn an average trajectory rather than multiple possible future paths.

To address these problems, Gupta et al. proposed Social GAN, which uses generative adversarial learning to predict diverse and socially acceptable pedestrian trajectories. The method also introduces a pooling mechanism to capture interactions between pedestrians and a variety loss to encourage different predictions for the same observed history <d-cite key="gupta2018social"></d-cite>.

### Methodology
Social GAN aims to predict the future trajectories of all pedestrians in a scene based on their observed movements. For pedestrian $i$, the observed trajectory is represented as 

$$
X_i = \{(x_i^t, y_i^t)\}_{t=1}^{t_{\mathrm{obs}}}
$$

where $(x_i^t,y_i^t)$ is the position at time $t$.

{% include figure.liquid 
   path="assets/img/2026-10-26-intent-trajectory-prediction/sgan_architecture.png" 
   class="img-fluid rounded z-depth-1" 
   caption="Figure 1: Social GAN architecture, including the generator, pooling module, and discriminator. Reproduced from Gupta et al. (2018), Figure 2." 
%}

The Generator uses an LSTM-based encoder-decoder architecture. The encoder processes the observed positions of each pedestrian and produces hidden states representing their movement history. The Pooling Module then combines information from pedestrians in the scene to capture their interactions. The decoder uses the encoded history, pooled information, and a random noise vector to generate future positions. By sampling different noise vectors, the generator can produce different future trajectories for the same observed history.

The proposed pooling mechanism considers the relative positions and hidden states of all pedestrians instead of only using a fixed local neighborhood. These features are processed by a multilayer perceptron (MLP) and combined using max pooling. This produces a fixed-size representation that summarizes information from other pedestrians and helps the model learn social interactions.

The Discriminator is another LSTM-based network. It receives a complete trajectory consisting of the observed history and either the real future path or a generated prediction. It then classifies whether the trajectory is real or generated. Through adversarial training, the generator learns to produce trajectories that are more similar to observed pedestrian movements.

To improve prediction diversity, the authors introduce a variety loss. For each observed history, the generator produces $K$ possible future trajectories using different random noise samples. The loss is calculated using the generated trajectory closest to the ground truth:

$$
\mathcal{L}_{\mathrm{variety}}=
\min_{k \in \{1, \ldots,K\}}
\left\|Y_i-\hat{Y}_i^{(k)}\right\|_2.
$$

Here, $Y_i$ is the ground-truth future trajectory, $\hat{Y}_i^{(k)}$ is the $k$-th generated trajectory, and $K$ is the number of sampled predictions. This loss encourages the model to generate different possible trajectories rather than making every prediction close to one average path. The variety loss is combined with adversarial training to improve both diversity and prediction accuracy.

### Experiments and Results

The authors evaluated Social GAN using the ETH and UCY pedestrian trajectory datasets, including five subsets: ETH, HOTEL, UNIV, ZARA1, and ZARA2. These datasets contain real pedestrian movements in different environments. The model observes 8 time steps (3.2 seconds) and predicts the next 8 or 12 time steps (3.2 or 4.8 seconds). The experiments use a leave-one-out setting, where four subsets are used for training and the remaining subset is used for testing.

The prediction accuracy is evaluated using two metrics:

- **Average Displacement Error (ADE):** The average Euclidean distance between predicted and ground-truth positions across all future time steps.
- **Final Displacement Error (FDE):** The Euclidean distance between the predicted final position and the actual final position.

Lower ADE and FDE values indicate more accurate predictions.

The authors compared Social GAN with several baseline methods, including Linear prediction, LSTM, and Social LSTM. Table 1 presents the ADE and FDE results on the five pedestrian datasets.

{% include figure.liquid
   path="assets/img/2026-10-26-intent-trajectory-prediction/sgan_results.png"
   class="img-fluid rounded z-depth-1"
   caption="Table 1: Quantitative comparison of trajectory prediction methods using ADE and FDE. Reproduced from Gupta et al. (2018), Table 1."
%}

The results show that SGAN-20V-20 achieves the lowest average ADE and FDE among the selected methods. The full model with pooling, SGAN-20VP-20, has slightly higher errors, but still performs better than the LSTM and Social LSTM baselines on average. The Social GAN results are based on selecting the best prediction from 20 generated samples during evaluation, while the baseline models produce a single prediction.

The authors also conducted ablation experiments to study the effects of variety loss and social pooling. The results show that variety loss improves prediction accuracy by encouraging the model to generate more diverse possible futures. Simply generating more samples from a model trained without variety loss does not provide the same improvement.

Although pooling does not produce the lowest displacement errors, it helps the model generate more socially acceptable trajectories. Figure 2 compares Social GAN with and without pooling in different pedestrian interaction scenarios.

{% include figure.liquid
   path="assets/img/2026-10-26-intent-trajectory-prediction/sgan_collision_avoidance.png"
   class="img-fluid rounded z-depth-1"
   caption="Figure 2: Comparison of Social GAN predictions without pooling (top) and with pooling (bottom) in collision avoidance scenarios. Reproduced from Gupta et al. (2018), Figure 5."
%}

The qualitative results show that the pooling-based model can better represent behaviors such as changing direction to avoid other pedestrians, moving around a group, and slowing down to let another person pass. The authors also report that the pooling-based model is approximately 16 times faster than Social LSTM in their benchmark, mainly because it avoids performing the same expensive pooling operations at every time step.

### Limitations and Takeaways

Social GAN shows that pedestrian trajectory prediction can benefit from modeling both social interactions and multiple possible future movements. Its main contributions include the GAN-based prediction framework, the pooling mechanism for capturing interactions, and the variety loss for generating diverse trajectories.

However, the method mainly focuses on pedestrian movement and does not explicitly model environmental obstacles, scene semantics, or long-term human goals. Its prediction performance also depends on the observed trajectories of surrounding pedestrians, which may be incomplete or noisy in real-world applications.

Another limitation is the best-of-multiple-samples evaluation. Although generating several possible trajectories improves ADE and FDE, a robot cannot know in advance which prediction will match the actual future movement. Therefore, additional planning methods would be needed to use these predictions safely in real robot navigation.

Overall, Social GAN provides an approach for predicting multiple plausible pedestrian trajectories instead of relying on a single future path. It also shows why modeling social interactions is important for robots navigating in environments shared with humans.

## Estimating Cognitive Biases with Attention-Aware Inverse Planning

### Motivation and Problem

Understanding human behavior is important for robots and autonomous systems that interact with people. However, people do not always make optimal decisions, even when they know what they want to achieve. One reason is that human attention is limited. For example, a driver may focus on the car directly ahead but fail to notice an obstacle on the road. The resulting driving behavior may look irrational, even though the driver is still trying to reach the destination safely.

Traditional inverse reinforcement learning (IRL) tries to infer an agent's reward function from observed behavior. However, these methods usually assume that agents make decisions using a complete representation of their environment. This assumption can be problematic when a person makes a mistake because they did not notice something, rather than because they preferred a risky action.

Banerjee et al. (2025) proposed Attention-Aware Inverse Planning (AAIP) to address this problem. Instead of explaining all behavior through reward preferences, the method models how people select information when making decisions. The main goal is to infer an agent's hidden attentional biases from their observed actions <d-cite key="banerjee2025estimating"></d-cite>.

### Methodology

The method builds on Value-Guided Construal (VGC), a framework that describes how an agent forms a simplified representation of the environment when planning. Rather than considering every object, the agent selects a subset of objects that are important for making decisions. This subset is called a **construal**, represented by $C$.

For example, when driving on a road with several vehicles and obstacles, a driver may only pay attention to the vehicles that are directly relevant to the current task. This reduces the amount of information the driver needs to process, but it may also lead to unsafe decisions if an important object is ignored.

In VGC, the agent balances the usefulness of a simplified representation against the cost of considering more objects. This is described by the Value of Representation (VOR):

$$
VOR(s,C) = V(s,\pi_C) + Cost(C)
$$

Here, $s$ is the environment state, $\pi_C$ is the policy based on the selected construal, and $Cost(C)=-\lvert C \rvert$ is a negative complexity penalty for the number of represented objects, as defined in the paper. Thus, adding this term penalizes larger construals rather than rewarding cognitive effort. The agent prefers a construal that supports effective decisions without requiring too much attention.

The authors extend this framework by adding a parameterized heuristic bias function, $H_\lambda(s,C)$. These biases represent an agent's tendency to attend to or ignore certain objects based on factors such as prior experience or visual salience. The probability of choosing a construal is modeled as

$$
\pi_\lambda(C \mid s)
\propto
\exp\left(VOR(s,C)+H_\lambda(s,C)\right).
$$

The parameter $\lambda$ controls the attentional biases. For example, a driver who is less likely to notice ice on the road may have a negative attention weight associated with ice patches.

Figure 3 illustrates how different attention biases can change an agent's driving trajectories, even when the task and reward function remain the same.

{% include figure.liquid
   path="assets/img/2026-10-26-intent-trajectory-prediction/aaip_attention_bias.png"
   class="img-fluid rounded z-depth-1"
   caption="Figure 3: Driving trajectories under different attentional biases. Reproduced from Banerjee et al. (2025), Figure 1."
%}

The main task of AAIP is to infer the hidden bias parameters from observed behavior. Given a state-action trajectory $\zeta$, the method uses maximum-likelihood estimation to find the parameters that best explain the observed actions:

$$
\lambda^*=\arg\max_\lambda P(\zeta \mid \lambda).
$$

Because the agent's construal is not directly observed, the likelihood is calculated by considering different possible construals:

$$
P(\zeta \mid \lambda)=\sum_C P(\zeta \mid C)\pi_ \lambda(C \mid s_1).
$$

Here, $P(\zeta \mid C)$ measures how well a particular construal explains the observed trajectory, while $\pi_ \lambda(C \mid s_1)$ represents the probability of selecting that construal at the beginning of the episode.

For more complex environments, calculating a separate optimal policy for every construal would be expensive. The authors therefore use a pre-trained Proximal Policy Optimization (PPO) policy and mask the observations so that only selected objects are visible. This allows one policy network to approximate decisions under different construals and makes the approach more practical for driving simulations.

### Experiments and Results

The authors evaluated AAIP in two environments: DrivingWorld and GPUDrive. Both experiments use simulated agents with known attentional biases, allowing the researchers to compare the estimated parameters with the true values.

The first experiment uses **DrivingWorld**, a simplified driving environment containing parked cars, ice patches, traffic cones, and a goal location. The agent must move toward the goal while avoiding obstacles. Three attentional bias parameters are considered, corresponding to ice, cones, and parked cars.

The authors generated behavioral data from 1,000 simulated agents with different bias weights. Each agent produced 125 trajectories across 25 scenarios. The model then estimated the three bias parameters from the observed trajectories.

Figure 4 compares the true bias weights with the estimated values.

{% include figure.liquid
   path="assets/img/2026-10-26-intent-trajectory-prediction/aaip_drivingworld_results.png"
   class="img-fluid rounded z-depth-1"
   caption="Figure 4: Recovery of attentional bias weights in DrivingWorld. Reproduced from Banerjee et al. (2025), Figure 2."
%}

The results show that AAIP can recover some attentional biases from behavior. The reported $R^2$ values are 0.77 for ice, 0.76 for cones, and 0.37 for parked cars. The lower result for parked cars suggests that some biases are more difficult to identify because avoiding parked cars is already strongly related to the task objective.

The authors also compare AAIP with traditional IRL. In an experiment with an agent biased to ignore ice, AAIP achieves a negative log-likelihood (NLL) of 265, while the tested IRL models achieve values between 513 and 537. Lower NLL indicates a better fit to the observed behavior. This comparison shows that changing reward parameters may not fully explain behavior caused by limited attention.

The second experiment uses **GPUDrive**, a driving simulator with scenarios selected from the Waymo Open Dataset. The experiments include situations such as lane changes, merging, and yielding to other vehicles. The authors use a pre-trained PPO policy to simulate driving behavior under different attentional representations.

In this environment, the authors generated data from 215 simulated agents, with 80 trajectories per agent across 10 driving scenarios. The attention model uses three heuristic features based on relative vehicle position, heading, and motion.

Figure 5 shows the results of estimating the attentional bias weights in GPUDrive.

{% include figure.liquid
   path="assets/img/2026-10-26-intent-trajectory-prediction/aaip_gpudrive_results.png"
   class="img-fluid rounded z-depth-1"
   caption="Figure 5: Recovery of attentional bias weights in GPUDrive using Waymo driving scenarios. Reproduced from Banerjee et al. (2025), Figure 5."
%}

The results show positive relationships between the true and estimated weights for all three heuristics, with $R^2$ values of 0.77, 0.76, and 0.83. These results suggest that AAIP can recover attentional bias parameters in more complex driving environments, not only simplified tabular tasks.

### Limitations and Takeaways

AAIP provides a different way to understand human behavior by explicitly modeling limited attention. Unlike standard IRL, which mainly explains behavior through reward functions, AAIP considers what information an agent may have attended to when making decisions. This can help explain why a person sometimes behaves in a way that appears suboptimal.

However, the experiments are based on synthetic agents rather than actual human drivers. Although GPUDrive uses real-world driving scenarios, the behavior used for bias estimation is generated by simulated policies. Therefore, the results do not yet demonstrate that the method can accurately identify attentional biases in real human behavior.

The method also assumes known task rewards and predefined attention heuristics. In real environments, people may change their attention during an interaction, while the current model keeps the selected construal fixed throughout an episode. In addition, using a masked pre-trained PPO policy may not always produce optimal behavior under each simplified representation.

Overall, AAIP shows that understanding human behavior requires considering not only what people want to achieve, but also what information they use when making decisions. For human-robot interaction, this perspective may help autonomous systems better predict and respond to behaviors caused by limited attention.

## Comparison and Discussion

Social GAN focuses on predicting where pedestrians may move by learning social interactions from trajectory data. In contrast, Attention-Aware Inverse Planning (AAIP) tries to explain why people behave in certain ways by modeling their limited attention. Social GAN can generate multiple possible future trajectories, while AAIP provides a more interpretable explanation of behavior through attentional biases.

During the class discussion, one question was how to combine individual pedestrian interactions with global crowd information. The class discussed using attention-based models, such as Transformers, to capture interactions across a scene. However, the professor pointed out that simply including more information may not be enough. Selecting relevant information is also important, especially when interaction data is limited. This also suggests a possible connection between the two papers, where attention modeling could help identify which surrounding agents are most relevant for trajectory prediction.

Another discussion focused on multimodal prediction and generalization. The professor mentioned that generating multiple trajectories does not guarantee that rare behaviors will be captured, since models still depend on the distribution of their training data. This helps explain why the variety loss in Social GAN is useful. The class also raised concerns about whether models trained on limited pedestrian datasets would generalize to different environments, crowded scenes, or people with different movement patterns.

For AAIP, the discussion focused on how attention may change over time and how inferred biases could improve safety. The current method assumes a fixed construal during an episode, but people may shift their attention as situations change. Examples discussed in class included driver warning systems and assistive robots that adapt to a person's behavior. A related challenge is distinguishing between someone ignoring a hazard and knowingly accepting its risk.

Overall, the discussion highlighted that predicting common behaviors is not enough for safe human-robot interaction. Rare or unexpected behaviors may be especially important, even when they are not well represented in training data. Combining trajectory prediction with attention-aware modeling could be an interesting direction for future research.

