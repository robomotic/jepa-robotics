
In the AI research of 2026, we are essentially witnessing the **"Great Convergence"** where 1960s Control Theory has finally married Deep Learning. Yann LeCun and other pioneers frequently point out that the "backprop through time" (BPTT) we use to train RNNs and world models is mathematically identical to the **Adjoint State Method** developed by Bryson, Ho, and others in the early '60s.

Here is why this 60-year-old math is suddenly the cutting edge of 2026:


**1\. The Rebranding: From "Plant" to "World Model"**

In the 1960s, control theorists talked about the **Plant** (the system being controlled) and the **Cost Function**. In JEPA/MPC, we’ve just swapped the vocabulary:

* **1960s:** Plant Dynamics $\\rightarrow$ **2026:** JEPA Predictor (World Model).  
* **1960s:** Adjoint State $\\rightarrow$ **2026:** Gradients backpropagated through the predictor.  
* **1960s:** Optimal Control $\\rightarrow$ **2026:** Model Predictive Control (MPC) / Planning.

### **2\. Why it’s "The New Black" Now**

If the math existed in 1962, why did we spend decades on Reinforcement Learning (RL) and Generative AI first?

* **The Latent Space Breakthrough:** Classical control required you to have a mathematical model of the world (e.g., $F=ma$). If you tried to apply it to a camera feed, the "state" was millions of pixels, and the math broke. **JEPA** solved this by encoding the camera feed into a **Latent Space**. We now do "Optimal Control" on abstract vectors, not raw pixels.

* **Self-Supervised Learning (SSL):** In the 60s, you had to hand-write the dynamics. Today, the JEPA Predictor *learns* the physics of the world just by watching video. This is the "secret sauce" that turned an engineering tool into "Autonomous Intelligence."

### **3\. The Shift from "System 1" to "System 2"**

The industry is currently pivoting away from **Autoregressive LLMs** (which are "reactive" or "System 1") toward **JEPA+MPC** agents (which are "deliberative" or "System 2").

* **Generative AI (Old Black):** Predicts the next token/pixel. It's a "fantasizer."  
* **JEPA+MPC (New Black):** Predicts the next *state representation* and checks it against an **Energy-Based Model (EBM)** to see if it makes sense. It's a "planner."

### **4\. Backprop Through Time (BPTT) is the Optimizer**

When you see an MPC planner "searching" for an action sequence, it is often literally performing gradient descent through the world model to minimize energy. In the 1960s, they called this "iteratively solving the adjoint equations." In 2026, we call it **"Planning as Inference."**

**Comparison: 1960s vs. 2026**

| Feature | 1960s Optimal Control | 2026 JEPA \+ MPC |
| :---- | :---- | :---- |
| **Model** | Hand-crafted differential equations. | Self-supervised latent predictor. |
| **State** | Low-dimensional physical variables ($x, v, a$). | High-dimensional embeddings ($z$). |
| **Optimization** | Adjoint Method / Pontryagin’s Principle. | BPTT / Differentiable MPC. |
| **Input** | Clean sensor data. | Messy, high-entropy video/audio. |

### **The comeback?**

We aren't really inventing new math; we are finally giving 1960s math the **computational power** and **representation learning** it needed to handle the real world.
