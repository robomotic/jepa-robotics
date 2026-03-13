# JEPA and Causality

To analyze a **humanoid robot** through the lens of Judea Pearl’s **Ladder of Causation**, we need to examine what our JEPA “world model” actually knows versus what it merely predicts.

Here is how the JEPA architecture maps to the Ladder and where the “Counterfactual Wall” appears.

### Rung 1: Association (Seeing)

- **JEPA status:** **Highly advanced.**
- **In our humanoid:** This is our JEPA encoder. It observes the robot walking and learns that “when the knee angle is \(X\), the foot is usually at height \(Y\).”
- **Shortcoming:** This is purely statistical. If the robot sees a person trip, it associates “trip” with “falling,” but it does not necessarily understand the physical *necessity* of the fall—only the correlation.

### Rung 2: Intervention (Doing)

- **JEPA status:** **Native capability.**
- **In our humanoid:** This is our **MPC (Model Predictive Control)**. We feed an action \(a_t\) (e.g., “apply 50Nm torque to the ankle”) into the predictor, and it outputs the next latent state \(z_{t+1}\).
- **Shortcoming:** While MPC “does” things, it predicts only the **forward** result of an intervention. It answers *“What will happen if we do X?”* but does not represent the underlying causal structure. If the robot fails to stand, our JEPA/MPC cannot determine whether failure came from weak motor output or a slippery floor—it only knows the resulting state was “lying down.”

### Rung 3: Counterfactuals (Imagining)

- **JEPA status:** **The missing link.**
- **The question:** *“I fell. Would I have stayed upright if I had moved my left foot 10cm farther back?”*
- **The problem:** To answer this, a model needs **abduction**. It must:
  1. **Observe** the actual state (I fell).
  2. **Infer** hidden noise or unobserved factors (e.g., “the friction coefficient of this specific tile was lower than expected”).
  3. **Update** the world model with that hidden factor and **re-run** the past with a different action.

---

## Why we struggle at Ladder 3 (Counterfactuals) with JEPA

The fundamental shortcoming of JEPA (and most current deep learning) in a humanoid context is the **lack of Structural Causal Models (SCMs)**:

1. **Latent inseparability:** JEPA compresses everything into a single vector (\(z\)). In a humanoid setting, “motor torque,” “gravity,” and “floor friction” are entangled. For counterfactuals, we need to vary *only* friction while holding gravity fixed. In a dense latent vector, we cannot reliably find a friction-only “knob” without perturbing everything else.
2. **Deterministic vs. probabilistic noise:** Pearl’s Rung 3 requires handling \(U\) variables (unobserved background factors). JEPA is often trained to be **deterministic** in latent space to avoid collapse. Because it does not explicitly model unseen causes of failure, it cannot “undo” them to imagine an alternative past.
3. **The Causal-JEPA frontier:** Research from **February 2026** (e.g., *C-JEPA: Learning World Models through Object-Level Latent Interventions*) attempts to address this by using **object-level masking** to separate physical entities. This enables **latent interventions** that are much closer to counterfactual reasoning.

### Example: The slipping humanoid

- **MPC (Rung 2):** “If I step here, I might slip. I’ll step there instead.” (Prediction)
- **Counterfactual (Rung 3):** “I slipped. If the floor had not been wet, my original step would have worked. Therefore, I do not need to change my walking style globally; I need to detect wet floors.” (Diagnosis/Learning)

**Shortcoming:** Our JEPA agent will likely learn to “walk differently everywhere” to avoid slipping, while a true Rung 3 agent isolates the specific *cause* and adapts only when that cause is present.

Building on Pearl’s Ladder, moving from **Rung 2 (Intervention)** to **Rung 3 (Counterfactuals)** in humanoid robotics requires shifting from “forward prediction” to “retrospective reasoning.”

In 2026, the leading direction for this shift is **disentangled causal latent spaces**, especially architectures like **Causal-JEPA (C-JEPA)** introduced by Meta researchers (including Yann LeCun) in early 2026.

### Core idea: Object-level masking

In standard JEPA, latent vector \(z\) is a black-box summary of the whole scene. If the robot falls, we cannot tell whether \(z\) encodes a “weak motor” or a “slippery floor.”

**Disentangled C-JEPA** uses **object-centric slots**. Instead of one vector, the encoder outputs multiple slots:

- **Slot A:** Robot internal state (joint angles, torques)
- **Slot B:** Floor properties (friction, slope)
- **Slot C:** External obstacles

With **object-level masking**—predicting Slot A’s future while Slot B is hidden—the model is forced to learn the independent causal influence of floor conditions on robot behavior.

### How this enables Rung 3 (Counterfactuals)

To reach Rung 3, our humanoid must answer: *“I slipped. Would I have stayed upright if the floor (Slot B) had been dry instead of wet?”*

| Step | Standard JEPA (Rung 2) | Causal-JEPA (Rung 3) |
| :--- | :--- | :--- |
| **Abduction** | Not possible; “wetness” is fused into a blurred vector. | **Possible;** model can backtrack and localize error in the interaction between Slot A and Slot B. |
| **Action** | Tests only “What if I move my leg differently?” | Tests “What if the environment variable was different?” |
| **Conclusion** | “I should never walk like that again.” | “The movement was fine; floor condition was the anomaly.” |

### The C-JEPA breakthrough (2026)

Recent work (e.g., *Nam et al., Feb 2026*) reports a **20% absolute improvement** on counterfactual reasoning tasks.

- **Latent interventions:** By perturbing specific slots (e.g., changing friction in Slot B), the robot can simulate “what-if” scenarios without physical execution.
- **Computational efficiency:** Reasoning over slots (objects) rather than pixels or thousands of patches can use only **~1% of the latent features** of prior approaches, enabling real-time imagination on robot hardware.

### Why this matters for us

If we build a standard JEPA, our robot becomes a **highly efficient pilot** (Rung 2): reactive and predictive.

If we move to a **causally disentangled JEPA**, our robot becomes a **scientist** (Rung 3): able to diagnose *why* failure occurred. In complex home environments, this is the difference between a robot that remains stuck on a rug and one that concludes, “I only trip on this specific rug; I should adjust my gait only when I detect this texture.”
