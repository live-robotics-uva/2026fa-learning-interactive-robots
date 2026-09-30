---
layout: distill
title: "Foundation Models for Language"
description: "Leveraging Large Language Models (LLMs) for high-level robot task planning, commonsense reasoning, and instruction following."
date: 2026-09-21
future: true
htmlwidgets: true
ready: false

# Reporter team authors
authors:
  - name: "Grace Bergquist, Daniel Evans, Jessi Morris, Atharva Tilak, Megan Vu"

bibliography: 2026-09-21-foundation-models-for-language.bib

toc:
  - name: "Introduction"
  - name: "InstructGPT"
    subsections:
    - name: "Methods"
    - name: "Overview"
    - name: "Supervised Fine-Tuning"
    - name: "Reward Model"
    - name: "Proximal Policy Optimization"
    - name: "Evaluation"
    - name: "Limitations"
  - name: "Voyager"
    subsections:
    - name: "Motivation"
    - name: "Goal"
    - name: "Implementation"
    - name: "Methodology"
    - name: "Results"
    - name: "Ablation"
    - name: "Takeaways"
  - name: "Student Q&A"
---

## Introduction

**The main question:** How can large language models help robots learn, reason, and plan about the world around them, especially in increasingly complex and dynamic environments? Can they produce plans a robot can understand and execute?

### How Foundation Models Matter for Robotics
- Real-world environments are open-ended and often messy, making task-specific robot learning hard to generalize across domains 
- LLMs provide several key advantages:
  - They absorb broad world knowledge during pretraining 
  - They can interpret natural-language instructions and break down high-level tasks into simpler parts 

### Background Papers
- **GPT-3: “Language Models are Few-Shot Learners”**
  - Adapts to new tasks from in-context prompt examples, with no gradient updates or task-specific fine-tuning 
  - Gap: Text adaptation != grounded physical action; outputs aren’t necessarily grounded in the environment 
- **Chain-of-Thought: “Chain-of-Thought Prompting Elicits Reasoning in LLMs”** 
  - Prompting the model to reason step by step improves its performance on arithmetic, commonsense, and symbolic reasoning tasks 
  - Gap: A reasoning trace can still be wrong or physically non-executable 
- **Zero-Shot Planner: “Language Models as Zero-Shot Planners”**
  - LLMs can break high-level task descriptions into plans without additional training 
  - Gap: Semantically reasonable != executable 

### Motivation for Papers Discussed In Class
- Addressing the executability gap 
  - InstructGPT attempts to tackle model alignment to match outputs to user intention 
  - VOYAGER attempts to tackle grounding by closing the loop with environment feedback (what happens during execution gets fed back into the next attempt) and storing skills for later reuse 

  ---

## Aligning Large Language Models with Human Intent Using Reinforcement Learning with Human Feedback <d-cite key="ouyang_training_2022"></d-cite> {#instructgpt}

There is a fundamental mismatch between the objective of large, commercial LLMs and their actual loss function: while their purpose is "to follow user instructions helpfully and safely," their loss function encourages successful predictions of the next token in a sequence, which may not always be the most helpful or safest response. While external methods like system prompts allow for some control over the behavior of LLMs, the foundational disagreement between the way they are trained and their intended use may still lead to incorrect, toxic, or unhelpful responses: this is called "misalignment." InstructGPT is the product of a collection of methods designed to align LLMs with user intent, such that their behavior is helpful, honest, and harmless. 

### Methods
#### Overview

{% include figure.liquid 
   path="assets/img/2026-09-21-foundation-models-for-language/instruct-outline.png" 
   class="img-fluid rounded z-depth-1" 
   caption="Figure 1: The three steps of InstructGPT. Blue arrows indicate the flow of data used in training." 
%}

The creators of InstructGPT followed three primary steps in its construction. These steps are illustrated in Figure 1.

1. **Supervised Fine-Tuning (SFT):** Collect a new dataset of desired behavior labeled by humans. Fine-tune a pretrained LLM with supervised learning. InstructGPT fine-tunes a GPT-3 model.

2. **Reward Model (RM):** Train a version of the fine-tuned model to predict the quality of a response. Responses in the training data are scored by human labelers.

3. **Optimize with Proximal Policy Optimization (PPO):** Fine-tune the unaligned SFT model with reinforcement learning via the PPO algorithm, where the RM provides the reward which tunes the model.

#### Supervised Fine-Tuning
InstructGPT began with a pretrained GPT-3 model that had been optimized for next-token prediction, thus remaining unaligned with user intent. The authors collected a new dataset of 13,000 training prompts annotated with appropriate responses by human labelers, and this data was used to perform SFT on the pretrained model. In order to keep the evaluation of this technique valid, the authors separated their labelers into testing and training groups; moreover, they surveyed inter-annotator agreement rates, finding that both training and testing labelers agreed with each other above 70% of the time. It should be noted that this is an expensive technique, as it requires a lot of time from humans.

#### Reward Model
The reward model is a separate instance of GPT-3, fine-tuned with the dataset from step 1. It also includes an architectural change: instead of outputting a probability distribution across possible next tokens, the authors replace the output layer with a single scalar value. The output of the RM is a score indicating the alignment of the response with human intent. 

In order to train the RM, an additional dataset is collected. This time, instead of having humans write their own responses, the authors generated 4-9 responses per prompt from the SFT model, then had a human rank those responses from best to worst. The RM is optimized with the following loss function:

$$\text{loss}(\theta) = -\frac{1}{\binom{K}{2}} E_{(x, y_w, y_l) \sim D} \left[ \log \left( \sigma \left( r_\theta(x, y_w) - r_\theta(x, y_l) \right) \right) \right]$$

where $r_\theta(x,y)$ is the output of the RM and $y_w$ is the better response between $y_w, y_l$, as ranked by labelers.

#### Proximal Policy Optimization
Finally, InstructGPT is fine-tuned with reinforcement learning. In training, the authors provide the SFT GPT-3 model with a prompt, score its response with the RM, and then use PPO to update the model's weights. However, this method risks teaching InstructGPT to leverage response patterns that don't fit natural language but do provide favorable rewards. To mitigate this, the authors apply an additional penalty based on the KL-divergence of the output of InstructGPT from the output of the original SFT model. 


### Evaluation

{% include figure.liquid 
   path="assets/img/2026-09-21-foundation-models-for-language/instruct-results.png" 
   class="img-fluid rounded z-depth-1" 
   caption="Figure 2: Preference results of InstructGPT varients compared against baselines." 
%}

The authors evaluated InstructGPT using human preference on a held-out dataset of prompts. They compared their method to the following baselines:
1. GPT-3 without modification,
2. GPT-3 with an instruction prefix (i.e. system prompt),
3. GPT-3 with SFT but without RL,
4. and GPT-3 fine-tuned on public NLP datasets (FLAN, T0).

In general, responses generated by InstructGPT were preferred by humans in most cases. The largest InstructGPT model was preferred over GPT-3 with instructions $71\pm4%$ of the time, and even the small 1.3B model was preferred over 175B base GPT-3, emphasizing how model performance does not necessarily provide alignment. In particular, the results showed that InstructGPT improves on trustworthiness and toxicity; however, it does not make a significant improvement on bias. 

Importantly, the authors were able to overcome an "alignment tax" that degraded InstructGPT's performance on successfully completing tasks; but, why did this happen? By introducing a different objective, that is, alignment, the authors encouraged InstructGPT to change its weights to fit the RM, potentially at the cost of general intelligence. They were able to mitigate this problem by mixing pre-training gradients in with the reward from the RM, thus keeping InstructGPT performant throughout alignment. 

#### Limitations

The primary limitation of InstructGPT is in its human labelers. Obviously, this introduces a high cost to training, but it is also an imperfect attempt at human-alignment. The humans hired by the researchers to perform labeling tasks represent a very small sample of all the humans that would like to use LLMs: all of the biases held by these labelers will be written into the internals of InstructGPT. This problem is worsened by the fact that all the labelers came from the same labeling contractor, which could introduce hidden biases to the training data. The authors highlight this with a simple example: nearly all the labelers spoke English and nearly all the prompts in the data were English instructions, meaning that alignment may not carry over to other languages. 

The authors acknowledge that, while InstructGPT improves on base GPT-3, they still found examples of unsafe, toxic, or biased outputs. Moreover, there was an unintentional consequence of aligning InstructGPT to the intent of the user: when given prompts to be as biased as possible, InstructGPT tended to generate *more* toxic outputs than the baseline models. Thus, future work should consider how to align models while keeping their guardrails intact. 

---

## Voyager <d-cite key="wang_voyager_2023"></d-cite>

Voyager is the first LLM embodied lifelong learning agent, which is situated in the video game Minecraft and continuously explores, acquires skills, and makes discoveries without human intervention, using GPT-4 without any parameter tuning. 

### Motivation

**Can LLMs be used to continuously learn new skills, and remember previously learned skills to make progress without fine-tuning?**
- New learned behaviors should be stored and reused instead of solved again from scratch. 
- Minecraft has no fixed end-goal or predefined task sequence, requiring exploration and discovery, much like the real world. 
- Goals and planning are hierarchical; must complete several base goals [get stone, make crafting table, etc.] in order to craft a higher level goal [make pickaxe].

### Goal

- Build an agent that can continuously explore an open environment without human intervention or supervision (in Minecraft). 
- Continuously expand the agent's skill library: learn new skills and make increasingly complex discoveries. 

### Implementation

- Uses code as the action space (Mineflayer JavaScript API) rather than low-level motor commands. 
- GPT-4: is used for prompting and in-context learning, generally used as the base model for thinking, planning, and programming. 
- GPT-3.5: used for code explanation tasks. 

### Methodology

Automatic curriculum, skill library, and iterative prompting are three integrated concepts of Voyager that enable its open-ended lifelong learning goals.

1. **Automatic Curriculum**
  - Current state passed as context to GPT-4; GPT-4 dynamically proposes the agent's next appropriate task (progressively harder tasks). 
  - Based on exploration progress, current state, and previous task history (previously completed tasks). 
  - Inputs (all player inputs): inventory, environment and surroundings, nearby blocks + entities, biome, position, health, hunger, time. 
2. **Skill Library**
  - Writes and stores reusable skills: a repository of verified, successful code behaviors (actions stored as executable programs, via code generation). 
  - Stored in (key, value) pairs, where Key is a vector embedding of the skill description (generated by GPT-3.5 via text-embedding-ada-002), and Value is executable JavaScript program. 
  - For new tasks, performs semantic retrieval of the top-5 most relevant previously learned skills. 
  - Complex tasks are achieved by retrieving previously completed tasks and feeding them to GPT-4, which generates a new program using existing skills. 
3. **Iterative Prompting and feedback**
  - Environment Feedback: chat log/inventory changes; missing requirements (e.g. no sticks for crafting a pickaxe). 
  - Execution Errors: code errors and unsupported actions (compiler/syntax and runtime errors). 
  - Self-Verification: a separate GPT-4 critic agent checks whether the task actually succeeded / the program works. 

If a task fails, it receives critique and refinement: the code is improved using feedback and the agent tries again, so it can fix its own code. Once verified, the program is stored in the Skill Library. 

### Results

Voyager consistently completes more tasks than any other baseline (evaluated against ReAct, Reflexion, and AutoGPT), performs faster, and can complete tasks other agents can't. 
- Discovers 3.3x more unique items (63 items) and traverses 2.3x longer distances (covers more map area). 
- Tech Tree Mastery: unlocks wooden tools 15.3x faster, stone tools 8.5x faster, iron tools 6.4x faster, and is the only method to unlock diamond tools. 
- Zero-Shot Transfer: the skill library can be transferred to a new world as a plug-and-play asset to solve novel tasks and make progress faster (e.g., substantially improves AutoGPT performance when provided). 

### Ablations and Limitations 

**Ablations:**
- Removing the automatic curriculum decreases item discovery by 93% 
- Removing self-verification reduces discovery by 73%. 
- GPT-4 vs. GPT-3.5: GPT-4 code generation discovers 5.7x more items than GPT-3.5. 

**Limitations:**
- Text-only perception (requires human visual feedback/critique for 3D construction tasks like building houses/portals). 
- High API cost (GPT-4 is ~15x more expensive than GPT-3.5). 

### Takeaways
1. LLMs CAN adapt and reason. 
2. LLMs allow humans to be a more direct part of learning. 
3. LLMs offer a useful way to remember and reuse skills. 

---

## Student Q&A {student-q-a}

### The Three Steps of InstructGPT 

**Q: How do the results of InstructGPT change when removing any of the three steps?**

A: Because step three, proximal policy optimization, is predicated on step two, the reward model, those two steps must be considered together. In Figure 2, it can be seen that GPT-3 with SFT alone improves significantly over the other baselines, but that InstructGPT with all three steps is by far the most preferred by human labelers. Clearly, steps two and three are important, if not necessary, for InstructGPT's success.  

However, the authors do not compare InstructGPT to a model trained with reinforcement learning but without SFT. Because RL is significantly cheaper (the response scoring is automated), it is worth asking how important the SFT step is in the first place, but the authors do not speak about this in their paper. 

<br><br>

### Comparisons With Other Methods  
sinfo -o "%.15P %.10a %.10l %.10D %.6c %.8m %.10T %N %E"

**Q: What do the methods Voyager is compared against use for an LLM?** 

A: GPT-4 
<br><br>

**Q: Why does VOYAGER outperform AutoGPT? Why is Voyager without Skill Library still better than AutoGPT with Skill Library?**

A: VOYAGER integrates a dynamic automatic curriculum, long-term memory via the skill library, and an iterative self-verification feedback loop. AutoGPT lacks the auto-curriculum and skill library, causing it to stall on long-horizon tasks. AutoGPT generates instructions and sticks to them (more likely to fail); Voyager can update instructions based on the environment. 
<br><br>

### How Voyager Works

**Q: In VOYAGER, what is the trade-off for using code as action tokens?**

A: Code provides high-level temporal and compositional abstraction for execution rather than primitive motor commands, but relies heavily on an API controller (Mineflayer) and accurate code synthesis without syntax errors.
<br><br>

**Q: What is the cost of retrieving from the skill library?**

A: Vector embedding generation and vector database query overhead. Scalability is maintained by keeping only the top-5 retrieved skill key-values in the LLM prompt context. 
<br><br>

**Q: Are goals broken down into low-level sequential actions, or saved as high-level action packages?**

A: Sequentially.
<br><br>

**Q: If reaching a diamond tool is a sequence of plans, do we need LLMs + skill code + env feedback, or just high-level actions?**

A: High-level text plans alone fail during execution because real environments involve unexpected obstacles and errors. 
<br><br>

### How Voyager Fails

**Q: How does VOYAGER handle roadblocks?**

A: If program refinement fails after 4 code generation iterations, the task is marked as failed and the curriculum agent proposes a different objective or alternative path. 
<br><br>

**Q: How does Voyager deal with critical/catastrophic failure, like dying?**

A: The paper doesn't discuss this. Probably just record death and restart. 
