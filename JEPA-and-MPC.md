# What is the interplay between JEPA and MPC?

JEPA is often described as a **world model** rather than just an image classifier. The architectural similarity between JEPA and **Model Predictive Control (MPC)** is quite striking, particularly in how they both handle "planning" and "state."

JEPA is effectively a **differentiable world model** designed to function within the framework of an MPC-like system. Here is where they overlap:

### **1\. The Internal Model (The "World")**

In MPC, you have a mathematical model of the system (e.g., how a car moves when you turn the wheel).

* **MPC:** Uses a transition function $s\_{t+1} \= f(s\_t, a\_t)$ to predict future states based on current state and actions.

* **JEPA:** Uses a **Predictor** ($P$) to do the same in latent space. It predicts the next latent representation ($z\_{t+1}$) based on a current representation and a "latent action" or "transformation" variable.

### **2\. Planning in Latent Space (The "Cost" Function)**

MPC doesn't just predict; it optimizes. It looks at a horizon of future steps and chooses the path that minimizes a cost function (like staying in the center of the lane).

* **MPC:** Minimizes a cost over a sequence of future actions.  
* **JEPA:** Is designed to be "steerable." Because it predicts embeddings rather than pixels, an agent can use JEPA to simulate multiple future "scenarios" in high-level abstract space. It can then select the "latent path" that leads to the most desirable representation.

### **3\. Resilience to "Noise" (Focus on Relevant State)**

One of the biggest headaches in MPC is when your model gets bogged down in irrelevant sensor noise.

* **MPC:** Works best when you have a clean "state" (e.g., velocity, angle).  
* **JEPA:** Specifically acts as the **encoder** that strips away noise. By predicting only the *meaningful* embeddings (the latent state), it provides the MPC-like controller with a clean, low-dimensional signal to work with, ignoring things like flickering lights or moving leaves that don't affect the goal.

### **4\. Comparison Summary**

| Feature | MPC (Control Theory) | JEPA (Architecture) |
| :---- | :---- | :---- |
| **State** | Physical variables ($x, v, \\theta$) | Latent Embeddings ($z$) |
| **Prediction** | Transition Equations | Predictor Network |
| **Optimization** | Minimizing Cost/Error | Minimizing Prediction Error |
| **Domain** | Usually Low-Dim / Physical | High-Dim / Visual / Complex |

**The "Missing Piece"**

The main difference is that standard MPC usually assumes you *already have* the model. JEPA is the method used to **learn** that model from scratch just by observing the world.

In LeCun’s vision of **Autonomous Intelligence**, JEPA serves as the "World Model" component, which an MPC-style "Configurator/Planner" queries to decide what to do next.
