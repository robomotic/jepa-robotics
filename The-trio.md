
# The core components for autonomy

This diagram illustrates the hierarchical trio relationship between the four components you asked about, detailing how they function within a modern robotics control system.

![jepa1](https://github.com/user-attachments/assets/e76f0718-aad8-4895-8b32-959b241eb3f8)


### **Diagram Breakdown:**

* **JEPA ENCODER (Top Left, Blue):** This node acts as the system's perceptual center. It takes a messy 'SENSORY INPUT (s₀)' (like a camera image) and encodes it into a clean, abstract, low-dimensional 'LATENT STATE (z₀)'.  
* **JEPA PREDICTOR (Center Left, Orange):** The predictor functions as the 'WORLD MODEL'. It receives the current latent state ($z₀$) and a sequence of 'PROPOSED ACTIONS (a₀...aₙ)'. By simulating how the world evolves based on those actions, it outputs multiple 'PREDICTED FUTURE LATENT STATES (ŝ)'.  
* **EBM (ENERGY-BASED MODEL) (Center Right, Green):** The EBM acts as the system's critic or evaluator. It receives the predicted future states from the predictor and assigns an 'ENERGY' value to each. A small 3D graph within the node shows how the EBM evaluates compatibility, with 'Low Energy' states being 'Good/Likely' and 'High Energy' states being 'Bad/Unlikely.'  
* **MPC (MODEL PREDICTIVE CONTROL) (Bottom Right, Gray/Teal):** The MPC acts as the 'PLANNER'. Using the EBM’s evaluations, it searches the vast space of possible actions to find the sequence that minimizes the total energy over a future horizon. Once optimal actions are found, it outputs 'ACTUATOR COMMANDS' to move the robot.

The arrows highlight the information flow through the loop, showcasing the closed-loop control system where the results are fed back into sensory input.