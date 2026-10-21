---
layout: distill
title: "Policy, World Model Code Generation"
description: "Generative policies, world model rollouts, and using LLMs for robotic code generation (ProgPrompt, WorldCoder)."
date: 2026-10-12
future: true
htmlwidgets: true
ready: true

# Reporter team authors
authors:
  - name: "Lainey Gordon"
  - name: "William Smith"
  - name: "Wendy Zheng"
  - name: "Andrew Parkinson"

bibliography: 2026-10-12-policy-world-model-code-generation.bib

toc:
  - name: "Introduction"
  - name: "ProgPrompt"
    subsections:
      - name: "Classical and LLM Planners"
      - name: "The Grounding Gap"
      - name: "Problem Formulation"
      - name: "Method"
      - name: "Evaluation"
      - name: "Discussion"
  - name: "WorldCoder"
    subsections:
      - name: "Background"
      - name: "Contextual MDP"
      - name: "Constraints"
      - name: "Pipeline"
      - name: "Experiments"
      - name: "Conclusion"
  - name: "Applications and Limitations"
    subsections:
      - name: "Applications"
      - name: "Limitations"
  - name: "Questions and Answers"
    subsections:
      - name: "ProgPrompt Questions"
      - name: "WorldCoder Questions"
      - name: "Generic Questions"
---

## Introduction {#introduction}

Large language models (LLMs) are pre-trained on most of the internet data to form a strong baseline for applying its generality to various different problems. This allows LLMs to applied to robotics scenarios<d-cite key="zeng_large_2025"/> ranging from high level decision making to lower level controls. LLMs have strong potential to increase the reasoning capabilities of robots by having an LLM connect all of the observations together and produce a reasonable action<d-cite key="wang_actionreasoning_2026"/> for complex scenarios that are difficult to fully model.

However, there two major challenges for deploying LLMs into physical embodiments:

- LLMs are not pre-trained on robotic embodiments, how can an LLM reasonable know how to utilize the physical space and actions provided by the robot?
- LLMs have the ability to hallucinate, how can the risk of errors due to hallucinations be reduced and improve robustness?

This lecture explores two different approaches to address the challenges:

- **PROGPROMPT: Generating Situated Robot Task Plans using Large Language Models <d-cite key="singh_progprompt_2023"/>:** Instead of allowing an LLM to interact directly with low level controllers, the LLM is instructed to write a function based, API style code to control the robot. By allowing the LLM to stay in its primary domain, the results should be more consistent since the robot model has already been abstracted into a code object. Through guided examples and self verification, PROGPROMPT is able to complete tasks at a higher rate compared to vanilla LLM inference.
- **WorldCoder, a Model-Based LLM Agent: Building World Models by Writing Code and Interacting with the Environment <d-cite key="tang_worldcoder_2024"/>:** Instead of using LLMs to purely reason about complex scenarios, have the LLM construct a world model of the environment based on state and reward feedback. This allows the LLM to be able the reference the current world model instead of trying to memorize the states from the context. The constraints increase robustness since the LLM can query the world model to predict the transition function of the environment. The LLM then creates and optimizes a policy to achieve the task based on the world model. Both the policy and world model are updated when key metrics do not align with true observations from the environment.

---

## ProgPrompt {#progprompt}

**Motivation**: Household tasks require both commonsense world understanding as well as situated knowledge of the current environment. LLMs know a lot about the world, and the question becomes how to leverage this understanding in an embodied robot.

**ProgPrompt's** key idea is to introduce situated-awareness in LLM-based robot task planning by framing it as Python program generation. The LLM is prompted with pythonic code form specifications of actions, objects, and examples, and it completes the program for a new task.

### Classical and LLM Planners

**Classical Task Planning**: Task planning traditionally relies on symbolic search (e.g. PDDL, STRIPS) over predefined domain models, but it struggles to scale in open-world environments.

**LLMs as Planners**: LLMs (e.g. GPT-3, Codex) possess vast commonsense knowledge. Recent work has used natural language (NL) instructions for zero-shot and few-shot task decomposition.

### The Grounding Gap

> **LLMs generate plausible sounding text, but they lack physical embodiment and environmental awareness**

**API Mismatch**: The LLM proposes actions that the robot cannot physically perform.

**Object Hallucination**: The LLM references objects not in the current room.

**No State Feedback**: The open-loop plan generation cannot adapt when a precondition fails.

### Problem Formulation

The task planning is modeled as a tuple:

$$
\langle O, P, A, T, I, G, t \rangle
$$

- $$O$$: set of all objects available in the environment
- $$P$$: set of properties of the objects; informs object affordances
- $$A$$: set of executable actions, which depends on the current state $$s \in S$$
- $$T$$: transition model $$T: S \times A \rightarrow S$$
- $$I, G$$: initial and goal states
- $$t$$: high-level task description

### Method

{% include figure.liquid path="assets/img/2026-10-12-policy-world-model-code-generation/r2-p1-arch.png"
class="img-fluid rounded z-depth-1"
caption="Overview of ProgPrompt architecture highlighting the components of the prompting, generated function, state feedback, and execution."
%}

ProgPrompt utilizes a *PROMPT for Planning* and a *PROMPT for Feedback* to obtain and refine a generated plan from the LLM. Key componenets of this process are the provided environmental information and examples, prompt comments, and assertions/assertion checks, discussed in deatil below.

**Robot Plan Representation**

Robot plans are represented as Pythonic programs. The prompt is structured as Pythonic code, and the LLM completes it. The plan function consists of API calls to action primitives that take objects as arguments, summary comments, and assertions to track execution. Rather than scoring candidates or generating text, the LLM generates an entire executable plan program.

**Why Code?** The motivation is that LLMs are heavily pretrained on code, including many programming tutorials and lots of documentation on the web. Code carries strict syntax, functional constraints, and procedural logic that natural language lacks. NL can often be ambiguous, whereas the function call either does or does not match the API.

**Prompt Design**

The prompt is written as a Python file with 4 parts:

1. Action Primitives as Imports
2. Available Objects List
3. Example Tasks
4. Open-Ended Function Header

**Task Plan Generation & Execution**

{% include figure.liquid path="assets/img/2026-10-12-policy-world-model-code-generation/r2-p1-funcex.png"
class="img-fluid rounded z-depth-1"
caption="Pythonic ProgPrompt plan for put salmon in the microwave."
%}

Generated plan functions, as shown in the figure, contain the following key components:

- **API calls** to action primitives with objects as arguments
- **NL comments** to break down tasks, focus on immediate goals, and reduce repetitive output. This process encourages and mimics "chain of thought" style reasoning and prompting.
- **Assertions with recovery actions**, which check preconditions before acting and fix the state when a check fails.

### Evaluation

The authors evaluated ProgPrompt across virtual simulation and real-world experiment domains.

**Metrics**

- Success Rate (SR): the fraction of task executions that successfully achieved all task relevant goal conditions
- Executability (Exec): the fraction of actions generated in the plan that are executable by the agent in that environment
- Goal Conditions Recall (GCR): compares the final state achieved against the ground truth goal conditions

#### Simulation

**Simulation Setup**: The simulation was run in VirtualHone, a deterministic simulation platform for typical household activities.
<!-- , with the action space $$A=\{\text{grab, putin, putback, walk, find, open, close, switchon, switchoff, sit, standup}\}$$. They experiment with 3 environments, each with 115 unique object instances, and create a dataset of 70 tasks. Each result is averaged over 5 runs across 10 tasks. -->

**Simulation Process**:

1. Simulator extracts the scene state.
2. ProgPrompt builds the LLM Prompt.
3. The LLM process the prompt and outputs the completed function.
4. The interpreter processes the code line-by-line.
5. The Simulator executes via API calls.
6. Repeat until the program is finished.

{% include figure.liquid path="assets/img/2026-10-12-policy-world-model-code-generation/r2-p1-simtable-comparison.png"
class="img-fluid rounded z-depth-1"
%}

- ProgPrompt w/ GPT-3 outperformed the zero-shot planner baseline and LangPrompt on every metric.
- Codex achieved the highest overall performance since it was explicitly trained on programming language code.
- Davinci underperformed relative to GPT-3 likely due to prompt length constraints limiting task example quantity.
- Removing comments reduced performance across all metrics
- Including assertion checks and recovery steps improved performance.

{% include figure.liquid path="assets/img/2026-10-12-policy-world-model-code-generation/r2-p1-simtable-tasks.png"
class="img-fluid rounded z-depth-1"
%}

- Tasks similar to prompt examples have higher GCR.
- No longer task with 11-18 ground truth actions were fully completed.
- High executability does not guarantee high GCR, since some tasks have multiple appropriate foal states.

{% include figure.liquid path="assets/img/2026-10-12-policy-world-model-code-generation/r2-p1-simtable-scenes.png"
class="img-fluid rounded z-depth-1"
%}

- ProgPrompt transfers to new scenes with the object list appended after example tasks in the prompt.
- ProgPrompt can infer that cutlery refers to spoons and forks even when only used to refer to knives in examples.

#### Real-World Experiments

**Experiment Setup**: A Franka-Emika Panda robot with a parallel jaw gripper was used to perform pick-and-place operations.

**Experiment Process**:

1. VLID detects items in the environment.
2. ProgPrompt builds the LLM Prompt.
3. The LLM process the prompt and outputs the completed function.
4. The interpreter processes the code line-by-line and maps to the environment.
5. A low-level control stack executes the plan.
6. Repeat until the program is finished.

{% include figure.liquid path="assets/img/2026-10-12-policy-world-model-code-generation/r2-p1-exp.png"
class="img-fluid rounded z-depth-1"
caption="Robot plan execution rollout example on a sorting task."
%}

- The LLM recognizes the banana and strawberry as fruits.
- The LLM ignores the distractor object.

{% include figure.liquid path="assets/img/2026-10-12-policy-world-model-code-generation/r2-p1-exptable.png"
class="img-fluid rounded z-depth-1"
%}

- The only failures occurred on the sorting task.
- Failure causes included distractors as well as an unexpected gripper failure.

### Discussion

**ProgPrompt**: an LLM prompting scheme for robot task planning that leverages the strength of LLMs in commonsense reasoning and code understanding to prompt for executable plans as programs.

*Recommendation from authors*: utilize a program-like prompt for LLM-based task planning and execution.

---

## WorldCoder {#worldcoder}

WorldCoder uses an LLM to generate a world model in the form of Python code, then uses that world model to create a plan.

### Background

People can pick up a new game or device in a handful of tries. We have strong priors from similar things we've seen and are exceptionally good at transferring skills to new goals. This is akin to learning a **world model**: a transition function that predicts the next state given the current state and an action. LLMs have a ton of knowledge about the world baked into their weights.

#### Limitations of existing approaches

- **Deep model-based RL:** Has weak priors and high sample complexity. With no prior knowledge, it must try countless actions until something sticks, and the trained network is a black box that is difficult to inspect.
- **LLM agents:** Have strong priors and don't need environment interaction to start, but they never update a world model. Each action also requires a separate LLM call, so cost grows with episode length.
- **WorldCoder:** Has strong priors and low sample complexity. It learns a world model that is inherently readable, and once the model is learned it needs at most one LLM query per new goal.

LLMs know a lot about the world but can't efficiently search or plan. WorldCoder takes advantage of that knowledge without relying on the LLM for what it's bad at.

### Contextual MDP

A standard MDP has state, action, transition, and reward: ⟨S, A, T, R⟩. WorldCoder uses a **contextual MDP**, which has context, state, action, and model: ⟨C, S, A, M⟩. The context *c* is the natural-language goal, and M(c) gives an MDP for it. The transition function is shared across goals and doesn't depend on context, while the reward changes with the goal.

The LLM builds two Python programs:

- **T̂**: the learned transition function
- **R̂**: the learned reward model, which also returns a flag indicating whether the goal has been reached

### Constraints

- **φ₁ (Consistency):** T̂ and R̂ must match everything the agent has seen in the past.
- **φ₂ (Optimism):** The agent believes the goal is reachable, meaning there is a sequence of actions that reaches the goal with positive reward.

When either constraint is violated, the world model is updated. Optimism drives exploration, and consistency acts as guardrails.

### Pipeline

1. Receive a natural-language goal *c* and record the initial state.
2. At each step, if the model violates φ₁ or φ₂, call the LLM to refine the code.
3. Act: take a random action with 5% probability, otherwise plan with the current model.
4. Store the experience (s, a, r, s′, c, d) in the replay buffer and repeat.

The LLM is good at debugging, so showing it the previous code and the transitions it fails to explain lets it generate a better function. Because a new world model can be built by editing the previous one, knowledge transfers across environments.

### Experiments

#### Sokoban

{% include figure.liquid path="assets/img/2026-10-12-policy-world-model-code-generation/sokoban-results.png" class="img-fluid rounded z-depth-1"%}

- The agent builds a working model from its first ~50 actions and generalizes from 2-box to 3-box levels.
- ReAct solves only ~15% of basic levels, and deep RL needs over 1M steps for 2-box levels.
- WorldCoder needs about 400k tokens at the start, then no more LLM calls.
- A teleportation variant shows it isn't just relying on memorized Sokoban solutions from the internet.
- Optimism doesn't help much here because the reward is dense and random exploration finds it quickly.

#### Minigrid

{% include figure.liquid path="assets/img/2026-10-12-policy-world-model-code-generation/minigrid-results.png" class="img-fluid rounded z-depth-1"%}

- The environments test different kinds of transfer: empty, door/key, unlock, fetch, and unlock-pickup.
- Transfer speeds up learning.
- Optimism helps more here and lets WorldCoder solve the harder environments.
- Without optimism, it struggles to follow new instructions, so optimism also helps catch LLM mistakes.
- PPO needs 10⁴–10⁵ steps on easy levels and fails UnlockPickup even after 3×10⁸ steps.

#### AlfWorld

{% include figure.liquid path="assets/img/2026-10-12-policy-world-model-code-generation/alf-world-results.png" class="img-fluid rounded z-depth-1"%}

- A household robot planning domain.
- Uses MCTS with a BM25 text-similarity heuristic to handle sparse reward and long horizons.
- Solves the task in the first episode after around 20 exploratory steps, then makes small corrections in later episodes.
- Optimism is necessary for any non-zero performance.

### Conclusion

WorldCoder shows that an LLM works best as a programmer of world models rather than as a planner or simulator. By writing its knowledge as code, the agent learns from far fewer interactions than deep RL, avoids the per-action LLM cost of ReAct-style agents, and ends up with a model that humans can read and edit. The optimism constraint (φ₂) is what makes exploration goal-directed, and it matters most in sparse-reward settings like Minigrid and AlfWorld. The main limitations are that it assumes deterministic dynamics and needs symbolic, discrete states, so extending it to noisy, perception-heavy environments is the natural next step.

---

## Applications and Limitations {#applications-and-limitations}

Given the capabilities of ProgPrompt<d-cite key="singh_progprompt_2023"/> and WorldCoder<d-cite key="tang_worldcoder_2024"/> what are the current applications and limitations of these methods?

### Applications {#applications}

Both ProgPrompt and WorldCoder exploit the coding capabilities of LLMs to gain access to its vast knowledge while also restricting the possibility of hallucinations. Forcing the model to write in code provides a structured syntax with automatic validation, as code is easier to validate than natural language. Thus, these methods would be especially beneficial for any kind of tasks that involve non-conventional robots or unfamiliar environments.

LLMs are exceptionally good at generalization, recognizing and applying learned patterns to unseen tasks. However, this is also harmful if the LLM is used for more specialized tasks. In robotics, an LLM, without any constraints, may assume that the robot is a humanoid and give commands based on this assumption. But, if the robot is a robotic dog or even an aquatic robot, then the commands the model gives is not applicable. On the other hand, using the LLM via a code interface enables the model to only access commands the robot is able to execute, which gives ProgPrompt and WorldCoder the advantage over vanilla LLMs. These approaches can better manipulate the robot to complete the task. 

Another application is for unfamiliar tasks or environments (i.e., tasks or environments extremely different from those the LLM was trained on). In this scenario, the model's knowledge provides little benefit other than commonsense knowledge, meaning that it is possible that using the LLM for planning is closer to random. However, WorldCoder provides a simulated environment for the robot to learn a suitable policy, and ProgPrompt provides the executable actions the model can use to explore the environment and complete the task. For both approaches, having a code interface enables better adaptability of the robots to unseen environments. 

### Limitations {#limitations}

Although code provides more structured syntax to restrict model output and prevent hallucinations, there are some key limitations of this type of approach.

1. *Executability does not equate to better performance.* These approaches only provide an alternative communication interface (via code), but it does not necessarily translate to improved task performance. We can even argue that using these approaches increases the model capability requirements, as the model not only needs to understand code but also generate valid code that accomplishes the goal. Smaller language models might be suitable for natural language communication but fail to output valid code syntax.
2. *The possibility of hallucination is reduced, not eliminated.* Even if we restrict model output to only generate code, the model can still generate incomplete or incorrect code. The main benefit is that we can quickly filter out invalid code, such as using a compiler. However, there is also the chance that the model generates syntactically valid but semantically incorrect code, such as calling the wrong function or using the wrong variable.
3. *The code interface requires more human setup.* Both methods require a code API to communicate with the robot, which could restrict the applicability of these methods. Furthermore, prior to using LLMs, a human annotator would need to design a prompt that enables the LLM to understand the code API and use it to generate commands. This type of approach requires a lot of effort to ensure compatibility between the robot and the LLM.

---

## Questions and Answers {#questions-and-answers}

This section provides common questions related to the presented topics. *If questions ask to take a side, we provide answers to support both sides.*

### ProgPrompt Questions {#progprompt-questions}

Questions and answers related to ProgPrompt<d-cite key="singh_progprompt_2023"/>.

#### Q: Is code the right interface for robots, or just the one LLMs happen to be good at?

*Code is the right interface for robots:* Code is an excellent medium for LLMs to interact with robotic systems because, unlike generating raw plans or actions, the code can be analyzed and verified. Even though there is still no explainability for the LLM inference, the chain of thought can be roughly seen through the comments and code structure. The other benefit of code is its reusability because the LLM does not need to generate a primitive action for every step, the function just needs to be generated once and can be queried again. This allows the motion to be more predictable between steps and verifiable by humans offline.

*Code is not an appropriate interface:* By having the LLM write code for interacting with the robot, we rely on the LLM to not hallucinate feasible and safe actions that are actually not achievable or dangerous. A much stronger method would be to combine LLMs with a library of actions using dynamic motion primitives (DMP)<d-cite key="anarossi_keymps_2025"/>. Leveraging DMPs allows the LLM to stay physically grounded with the robot and environment by focusing on utilizing a set of motions to complete a high level task.

#### Q: ProgPrompt hit 90% executability but only 40% success with Codex. Is "executability" really a meaningful metric?

*Executability defines a useful intermediate state:* Executability is the first step in determining whether a task can be achieved because if the LLM creates a broken function then there is zero chance of the task being completed. This metric allows us to approximately determine how many high level logic mistakes were made based on the difference between executability and success rate ($90-40=50\%$). The metric also allows for targeted fixes either structurally, prompt design, examples, etc. since the system is getting stuck without making any progress.

*Executability does not capture anything meaningful:* The authors do not formally define executability which leaves the readers guessing what it actually represents. Assuming it means that the state checks through assertions are violated or the code is un-runnable, then the metric does not have a strong use in the analysis. It is is too broad of a metric to allow us to determine what went wrong during the process. Without more details we do not know why the LLM failed to generate the code. Maybe the LLM hallucinated an feasible action, the current state, objects in the scene, etc. but we do not know without finer metrics.

### WorldCoder Questions {#worldcoder-questions}

Questions and answers related to WorldCoder<d-cite key="tang_worldcoder_2024"/>.

#### Q: WorldCoder's "optimism" makes the agent believe the goal is reachable and test it. But is that safe to deploy on a real robot?

*Optimism is risky and unsafe for real deployments:* Being optimistic can cause the robot to move towards unsafe regions and attempt tasks that a reasonable policy would not propose. Even though safety filters such as control barrier functions<d-cite key="ames_control_2017"/> can be employed to guarantee safety, we should be designing our systems to be safe at every layer and not rely on a catch all at the end. So the policy should be grounded with a notion of safety and risk to prevent overly optimistic goals from pushing robot into situations that it should avoid. Maybe this is implemented as a direct foil to optimism with something like pessimism, where certain conditions cause the robot to be wary of unsafe scenarios.

*Optimism is conditionally safe for the real world:* Without constraints, overly optimistic goals can be risky but since WorldCoder can analyze rollouts based on the current world model then it should be able to determine whether a goal is unsafe before becoming unsafe. However, this relies on the world model having sufficient alignment and correlation with the real world. If there are parts of the world that are not modeled due to occlusions or sensing limitations then the policy could direct the robot to an unsafe goal. So given an accurately created world model then the optimism trait should not hinder robot safety.

#### Q: Coding agents already write and test software. Could they eventually build and debug robotic systems autonomously?

*Coding agents are not capable to build robotic systems:* Coding agents can do a lot of things very efficiently such as running test suites for pinning down specific errors and then proposing a patch. This is done iteratively until the task is complete. When the task is just coding this can be achieved but not guaranteed because LLMs have a tendency to scope creep<d-cite key="qu_overeager_2026"/> and might end up changing the problem constraints or objectives during the process. This is already an issue for solving an end to end problem but since robotic systems need physical embodiments, the coding agent needs access to those embodiments. Physical systems are notoriously difficult to work with and most require physically working with them to run experiments. Without testing a system on physical hardware, even in a lab setting, then the efficacy of the system is still unknown and not validated. The best chance a coding agent would have at building and debugging a whole robotic system is to leverage services that abstract the physical robots away from the user. The Robotarium<d-cite key="wilson_robotarium_2020"/> allows users to test code in simulation and then remotely deploy to physical robots in a managed environment. But this is just a setup for niche experiments and does not allow for real world experiments outside of the laboratory.

### Generic Questions {#generic-questions}

Questions and answers related to ProgPrompt<d-cite key="singh_progprompt_2023"/> and WorldCoder<d-cite key="tang_worldcoder_2024"/>.

#### Q: Neither approach updates the LLM itself. Is that a strength (interpretable, editable) or a restriction (ceiling)?

*In context learning is a strength:* Updating LLM weights is computationally costly and risky for shifting the baseline as the task progresses. The only way to reasonably try and update the LLM weights is with fine-tuning because the model is too big to fully train. Since fine-tuning focuses on the layers closest to the output layer, significant changes in outputs can occur from limited training examples. This is risky for model that is supposed to be used in robotic systems where the environment can change quickly. Just because the weights are not updated does not mean the model does not learn and adapt to the prompts it has been given. In context learning<d-cite key="dong_survey_2024"/> allows the model to adjust its inference by holding information in its context window. This is an effective method for allowing the model to "learn" the preferences or requirements for the tasks.

*Not updating the weights is restricting:* Without updating weights the LLM is relying on its context to learn how to respond to a task it has previously encountered. The LLM can not just be blindly finetuned with the new experiences and must go through an algorithm designed to handle streams of new data for fine-tuning<d-cite key="liu_sample_2026"/>. If such a method is implemented and does not take too much compute resources, then it will be beneficial to train the new learned concepts into the model for permanent reuse.
