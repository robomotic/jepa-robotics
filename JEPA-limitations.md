While the JEPA (Joint-Embedding Predictive Architecture) is a powerful "World Model" framework, it isn't a magic bullet. As of 2026, researchers (including Yann LeCun at his new **AMI Labs**) have identified several "walls" this architecture hits.

Here are the primary limitations of the current JEPA+MPC approach:

**1\. The Temporal Horizon & Granularity Gap**

JEPA is excellent at "short-term imagination" (e.g., *if I push this, it moves*), but it struggles with **Long-Horizon Planning**.

* **The "Blurring" Effect:** Just as humans can’t perfectly imagine every turn of a 100-mile drive, the JEPA Predictor’s latent states become "blurry" or accumulate errors as you project further into the future.  
* **Granularity Mismatch:** High-level tasks (e.g., "Make a sandwich") require **Hierarchical JEPA**. You need one JEPA for "moving fingers" (millisecond granularity) and another for "following a recipe" (minute-long granularity). Currently, connecting these layers without the top layer "forgetting" the bottom layer's constraints is a major engineering challenge.

### **2\. The "Short-Cut" & Latent Collapse Problem**

Even with regularization (like VICReg), JEPA models sometimes find "cheats" during training:

* **Information Leakage:** If the masking isn't aggressive enough, the model can "see" the answer through the edges of the mask. Instead of learning physics, it learns to be a very fancy copy-paster.  
* **Feature Bias:** Theoretical analysis shows JEPAs have a "slow feature bias." They tend to ignore fast-moving, high-frequency details (like a bouncing ball's exact impact) in favor of slow, stable features (the wall behind the ball). This is great for high-level "concepts" but dangerous for high-speed safety tasks.

### **3\. Continuous Learning (Catastrophic Forgetting)**

JEPA models are usually trained in large "batches" on internet videos.

* **Adaptation:** If you take a JEPA robot from a sunny kitchen to a dark warehouse, it may struggle to adapt its latent representations in real-time.  
* **The Stability-Plasticity Dilemma:** If you let the model learn *too* fast from new data, it might "forget" basic physics it learned previously (like gravity). Currently, maintaining a "permanent" world model while allowing for "instant" learning is an unsolved trade-off.

### **4\. Safety & Formal Guarantees**

Because JEPA operates in an **opaque latent space**, it is a "Black Box" for safety.

* **Unpredictable "Dead Zones":** In MPC, if your model enters a part of the latent space it hasn't seen before, its energy predictions become garbage. The robot might suddenly "hallucinate" that a cliff is a flat floor.  
* **Lack of Formal Proofs:** Unlike traditional control theory (which uses math to *prove* a robot won't hit a human), JEPA relies on **statistical probability**. For mission-critical tasks (surgery, autonomous flight), we still lack the mathematical tools to "guarantee" a JEPA-based agent won't take a high-energy, dangerous action.


**Comparison of Current Challenges**

| Limitation | Impact on Robot | 2026 Research Direction |
| :---- | :---- | :---- |
| **Temporal Horizon** | Forgets the "big picture" goal. | **Hierarchical JEPAs** (stacked layers). |
| **Unpredictability** | Can't handle the "flickering TV" perfectly. | **Discrete-JEPA** (symbolic reasoning). |
| **Safety** | No mathematical guarantee of "no-crash." | **Hybrid-Systems** (JEPA \+ Symbolic Rules). |
| **Learning** | Forgets old skills when learning new ones. | **Bayesian JEPA** (probabilistic priors). |

### **Takeway**

JEPA is essentially a **"Intuitive Physics Engine."** It gives robots "common sense," but it doesn't yet give them "long-term logic" or "certified safety."
