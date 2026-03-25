# JEPA, Causality, and the Pearlian SCM Framework: Toward Causal World Models in Atari Breakout

## Abstract

This paper explores the potential of Joint Embedding Predictive Architectures (JEPA) to develop causal reasoning capabilities, specifically mapping JEPA to the Pearlian Structural Causal Model (SCM) framework. Using the practical context of training a JEPA agent to play Atari Breakout, we assess whether JEPA can progress from learning mere associations (Ladder 1) to interventions (Ladder 2) and counterfactuals (Ladder 3) as defined in Pearl’s Causal Hierarchy. We analyze the strengths and limitations of JEPA in causal discovery, discuss recent advances, and outline the requirements for achieving higher-level causal reasoning in world models.

## 1. Introduction

### 1.1 Pearlian Causality and SCM
Pearl’s Structural Causal Model (SCM) framework provides a formal language for representing and reasoning about cause-effect relationships using Directed Acyclic Graphs (DAGs). The SCM framework distinguishes between three levels of causal reasoning:

- **Association (Ladder 1):** Observing statistical dependencies between variables.
- **Intervention (Ladder 2):** Understanding the effects of actions using the $do$-operator, which simulates interventions by "cutting" incoming edges in the DAG.
- **Counterfactuals (Ladder 3):** Reasoning about alternate realities ("What if?") by considering hypothetical interventions and their consequences.

In the context of Atari Breakout, a simplified DAG might look like:

1. **Action ($A$)** $\rightarrow$ **Paddle Position ($X_p$)**
2. **Paddle Position ($X_p$)** + **Ball State ($S_b$)** $\rightarrow$ **Collision Event ($C$)**
3. **Collision Event ($C$)** $\rightarrow$ **Future Ball State ($S'_b$)**

### 1.2 Joint Embedding Predictive Architectures (JEPA)
JEPA is a family of self-supervised world models that learn to predict future states in a latent space, conditioned on actions. Unlike pixel-level predictors, JEPA focuses on learning high-level, invariant representations that capture the controllable dynamics of an environment. Recent research has explored how JEPA can be extended to discover causal structure and perform interventions in complex environments.

Key features of JEPA include:
- **Action Conditioner:** Models the effect of actions on future states, analogous to the $do$-operator in SCM.
- **Latent Masking:** Enables the model to test for sufficiency and necessity of information, similar to controlling for variables in a DAG.
- **Slot Attention:** Encourages the model to discover object-centric latents, mapping to nodes in the SCM.

### 1.3 Motivation: JEPA in Atari Breakout
Atari Breakout provides a rich testbed for causal discovery: the agent must learn how its actions (moving the paddle) affect the ball, bricks, and ultimately the score. The challenge is to move beyond mere correlation and build a model that understands the true mechanisms of the game. The environment contains both controllable (paddle, ball) and uncontrollable (score, flickering pixels) elements, making it ideal for studying causal discovery and invariance.

## 2. Problem Statement

**Can JEPA, as currently formulated, develop the capacity to reason about causality at all three levels of Pearl’s Ladder? What are the requirements and limitations for achieving this in practice?**

## 3. Mapping JEPA to the SCM Framework in Breakout


### 3.1 Data Collection Strategies and Causal Discovery
The way data is collected fundamentally shapes the latent space and the causal understanding that JEPA can acquire. Consider four strategies for collecting training data in Breakout:

| Strategy | Effect on Data Distribution | Impact on World Model & Planning |
| :--- | :--- | :--- |
| **A) Expert Demonstrations** | Very narrow; only "optimal" trajectories. | **The "Brittle" Model:** The model only learns the physics of winning. If the agent makes a small mistake during planning, it enters a state it has never seen, and the latent predictions collapse because it doesn't know how "failure" works. |
| **B) Random Actions** | High entropy; lots of "moving air." | **The "Noisy" Model:** The model sees the paddle move, but the ball-paddle collision (the most important causal event) happens rarely. It takes a massive amount of data to learn the physics of the bounce. |
| **C) Static Paddle** | Minimal action-effect correlation. | **The "Passive" Model:** Since the paddle doesn't move, the model fails to learn the *action conditioner* part of JEPA. It learns ball physics but doesn't understand its own agency. |
| **D) Heuristic (Follow X)** | **Dense causal interactions.** | **The "Causal" Model:** By following the ball, you maximize the frequency of the "collision" event. This provides the JEPA model with a rich dataset of how its actions directly influence the ball’s trajectory. |

**Strategy D** (Heuristic Tracking) provides the best curriculum for JEPA, as it ensures frequent paddle-ball collisions and a rich set of causal interactions. However, perfect tracking can introduce spurious correlations, where the model might falsely infer that moving left always causes the ball to move left. To address this, "dithering" (adding noise to the heuristic) is necessary to break confounds and enable true causal discovery.

#### The "Latent Collapse" Risk
If the model never observes certain events (e.g., paddle-ball collisions), it may treat the ball as unpredictable noise and collapse its representation, focusing only on the paddle. This highlights the importance of data diversity and targeted interventions.

### 3.2 JEPA Mechanisms and Pearlian Concepts
- **Action Conditioner:** Analogous to the $do$-operator; enables interventions.
- **Latent Masking:** Tests for sufficiency/necessity, akin to controlling for variables in a DAG.
- **Slot Attention:** Discovers object-centric latents, mapping to nodes in the SCM.

### 3.3 Interventional Noise and the $do$-operator
Adding random noise to the agent’s policy (e.g., 10% random actions) simulates the $do$-operator by decoupling actions from observations, allowing the model to distinguish true causation from correlation.

### 3.4 Limits of Causal Identification in JEPA
While slots help discover variables, identifiability is limited by unobserved confounders and Markov equivalence. Without interventions, JEPA cannot resolve all causal directions, mirroring the limits of observational data in SCM.

## 4. Analysis: JEPA and the Causal Ladder


### 4.1 Ladder 1: Association
JEPA excels at learning statistical dependencies and associations in the environment. For example, it can learn that moving the paddle left results in the paddle appearing on the left side of the screen. However, without interventions, it may conflate correlation with causation, especially if the data collection strategy is biased.

JEPA’s invariance-seeking latent representations filter out exogenous noise (e.g., score counter, flickering pixels). The encoder learns to ignore features that are unpredictable or irrelevant for future prediction, focusing on the controllable dynamics of the environment.

### 4.2 Ladder 2: Intervention
Through action conditioning and interventional noise, JEPA can learn the effects of interventions. By adding random noise to the agent’s policy (e.g., 10% random actions), the model simulates the $do$-operator, decoupling actions from observations and allowing the model to distinguish true causation from correlation.

For example, by occasionally moving the paddle randomly instead of following the ball, the model observes cases where the ball’s trajectory is independent of the paddle’s movement, except at the point of collision. This enables the model to correctly assign causality to the paddle only when it actually influences the ball.

### 4.3 Ladder 3: Counterfactuals
Recent advances, such as Causal-JEPA and counterfactual masking, enable the model to perform "what-if" reasoning. By ablating object-centric latents (e.g., masking the paddle slot), the model can simulate alternate outcomes and test whether the ball would have bounced if the paddle had not been present. This approach approximates counterfactual reasoning, allowing the agent to plan and generalize to novel situations.

## 5. Discussion


### 5.1 Strengths
- **Alignment with Causal Discovery:** JEPA’s object-centric and action-conditioned design aligns naturally with causal discovery. The use of slot attention and latent masking enables the model to discover and test causal relationships between objects in the environment.
- **Practical Tools for Causal Hypotheses:** Interventional data collection and latent masking provide practical tools for testing causal hypotheses. By simulating interventions and counterfactuals, the model can validate its understanding of the environment’s causal structure.
- **Intrinsic Curiosity:** Intrinsic curiosity mechanisms drive the agent to explore and validate new causal links. The agent seeks to reduce epistemic uncertainty in its world model, leading to more robust and generalizable representations.

### 5.2 Limitations
- **Identifiability Limits:** Identifiability is fundamentally limited without interventions or prior knowledge. Unobserved confounders and Markov equivalence can prevent the model from correctly identifying causal directions.
- **Semantic Gap:** Mapping latent slots to semantic variables remains a challenge. Without structured latents, it is difficult to perform targeted ablations and counterfactual queries.
- **Experimental Design:** Experimental design for ablation and counterfactual queries requires structured latents and a way to map them to objects of interest. This remains an open problem in unsupervised representation learning.

### 5.3 Implications for Agent Design
- **Curriculum Learning:** Curriculum learning (Strategy D + noise) is essential for robust causal world models. The agent must experience a diverse set of interactions to learn the true causal structure of the environment.
- **Object-Centric Representations:** Object-centric representations (slots) are critical for interpretable and testable causal reasoning. They enable the model to perform targeted interventions and validate its causal hypotheses.
- **Invariance-Seeking Loss:** JEPA’s invariance-seeking loss prevents distraction by exogenous noise (solving the "Noisy TV" problem). The model learns to ignore features that are unpredictable or irrelevant for future prediction.

## 6. Conclusion

JEPA architectures, when combined with interventional data collection, object-centric latents, and curiosity-driven exploration, have the potential to climb Pearl’s Ladder of Causation. While current models can achieve association and intervention, true counterfactual reasoning requires advances in latent structure and experimental design. Mapping JEPA to the SCM framework provides a principled path toward causal world models capable of robust planning and generalization in complex environments like Atari Breakout.

Future work should focus on:
- Developing methods for mapping latent slots to semantic variables.
- Designing experiments for targeted ablations and counterfactual queries.
- Integrating intrinsic curiosity mechanisms that drive exploration of novel causal links.

Ultimately, the integration of JEPA with the Pearlian SCM framework represents a promising direction for building agents that can reason about and manipulate the causal structure of their environments, enabling robust planning, generalization, and transfer in complex tasks.

---

*References and further reading can be added for LaTeX/ArXiv submission.*
