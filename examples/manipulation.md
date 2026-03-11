# Robotic manipulation scenario

Imagine a **Franka robotic arm** (a standard 7-joint industrial arm) in a messy kitchen. It has been tasked with picking up a specific **blue mug** it has never seen before, surrounded by distracting objects like a bag of chips, a flickering TV in the background, and shifting sunlight.

![jepa2](https://github.com/user-attachments/assets/e7a3b346-f90b-44ef-9ec2-c833ce11bf35)

Here is how the **JEPA \+ EBM \+ MPC** trio works together to solve this:

### **1\. The JEPA Encoder (Seeing only what matters)**

Traditional robots get "distracted" by the bag of chips or the TV.

* **The JEPA Encoder** (like **V-JEPA 2**) takes the camera feed and compresses it into a "latent representation."  
* It ignores the flickering TV and the texture of the chips because they aren't "predictable" or relevant to the arm's movement.  
* **Result:** The robot "sees" an abstract state: *Arm at Position A, Blue Mug at Position B, Obstacle at Position C.*

### **2\. The JEPA Predictor (Imagining the future)**

Instead of moving physically, the robot "imagines" a sequence of actions in its head.

* **Input:** Current latent state \+ a proposed sequence of motor commands (e.g., "Rotate Joint 1 by 10°, lower Joint 3").  
* **Output:** A predicted **latent state** of where the arm and mug will be in 0.5 seconds.  
* **Crucial Difference:** It doesn't imagine pixels. It doesn't "draw" a picture of a arm; it just calculates the mathematical "thought" of where the arm will be.

### **3\. The Energy-Based Model (The Evaluator)**

Now the robot needs to know if its "imagined" future is good or bad.

* **The Energy Function** acts as the critic. It assigns a "score" to the predicted state.  
* **High Energy (Bad):** If the predicted state shows the arm's latent representation overlapping with the table's latent representation (a collision).  
* **Low Energy (Good):** If the predicted state shows the arm's gripper representation closer to the mug's representation.  
* **Regularization:** The EBM ensures the robot doesn't find a "cheat" (like a zero-energy state that means nothing) by pushing the energy of "unrealistic" states higher.

### **4\. The MPC Loop (The Decision Maker)**

This is where the actual "control" happens. The robot uses a method called **Shooting** or **Cross-Entropy Method (CEM)**:

1. **Sample:** The robot randomly generates 500 different possible "action sequences" (e.g., different ways to reach for the mug).  
2. **Predict:** It runs all 500 through the **JEPA Predictor**.  
3. **Score:** It uses the **EBM** to see which of those 500 paths has the lowest "Energy" (the best combination of reaching the goal and avoiding a crash).  
4. **Execute:** It takes the **first step** of the best sequence.  
5. **Repeat:** It looks at the camera again and repeats the whole process 30 times a second.

**Why this is a "Breakthrough"**

In 2025/2026, models like **V-JEPA 2-AC** (Action-Conditioned) have shown that they can do this **"Zero-Shot."** This means you can take a robot trained on internet videos of *humans* moving things, put it in a brand new lab it has never seen, and it can pick up the mug immediately. It doesn't need to practice 10,000 times because it already understands the **"Physics of the World"** through its internal JEPA world model.

### **1\. The Perceptual Layer (Top Row)**

The process begins with the raw sensory input (top left, "INPUT SCENE"), where the arm (A) sees a cluttered counter:

* The **JEPA Encoder** (top center) receives this image. It is specifically designed to strip away irrelevant noise (like the **Flickering TV** or rapid light shifts).  
* It outputs the **Current Latent State** ($s₀$) (top right), which is a clean, abstract mathematical representation. In this state, the robot knows the positions of the arm and the crisps but flags the **Mug as Unknown**.

### **2\. The Mental Model (Center Brain)**

The center of the diagram, depicted as a physical brain model, is the **Predictor (World Model)**. This is where the robot "imagines" potential future scenarios without moving physically:

* Because the blue mug is obscured, it cannot make a single, precise prediction. Instead, it uses a **Latent Variable ($z$)** (inputting from the center left).  
* $z$ acts as a placeholder for the hidden information (all the *possible* locations of the mug).  
* By varying $z$, the Predictor simulates three plausible paths: Path 1 (Move Left), Path 2 (Move Right), and Path 3 (Nudge Crisps).

### **3\. Executive Planning (MPC) (Right Box)**

The bottom right panel is where the final decision is calculated.

* The **Configurator** defines the task goal: *"Pick up the blue mug."*  
* The **Energy-Based Model (EBM)** acts as a critic. It scores the three imagined futures based on the Configurator’s goal.  
* The EBM energy landscape is warped so that the states representing **"Goal Reached"** (bottom right of the graph) have the **Lowest Energy** (labeled with the green 'Low E' bar). Paths 1 and 2 result in a 'Miss' and are assigned **High Energy**. Path 3 (Nudge Crisps) is unique; it is scored with Low Energy because it leads to an informative state: "Mug Visible."

### **4\. Selected Action (Bottom Right)**

The Model Predictive Control (MPC) planner receives all three energy scores:

* It **MPC Selects Lowest Energy Action**.  
* It chooses **Path 3** (Nudge Crisps).  
* It initiates the very first step of that sequence: **Selected Action $a₀$ (Start Nudge)**, which is sent back to the physical robot, restarting the perception loop until the mug is successfully retrieved.

By itself, JEPA is **objective-blind**. It just learns the "laws of physics"—how things move, fall, or obstruct each other. It doesn't inherently care about your coffee.

To make the robot pick up the mug, we have to "hook up" two extra components to the JEPA world model. This is where the **Task-Specific Cost** and the **Configurator** come in.


**1\. The Role of the "Configurator"**

In Yann LeCun’s architecture, there is a module called the **Configurator**. Think of this as the "Boss" that gives the robot a reason to live.

* **JEPA** says: "If I move my arm here, the mug moves there." (Passive knowledge)  
* **The Configurator** says: "Currently, the Energy of the state 'Mug in Hand' is **0**, and the Energy of 'Mug on Table' is **100**."

The Configurator manually **warps the Energy landscape** to make the "Mug" state the most desirable "Low Energy" point.

### **2\. How the Robot Distinguishes the Mug from the Crisps**

When the MPC (the planner) starts "imagining" futures using the JEPA predictor, it asks the **Energy-Based Model (EBM)** to score them.

* **Scenario A (Moving toward the Crisps):** JEPA predicts the arm will be near the chips. The EBM looks at the Configurator's goal ("Get the Mug") and says: *"This state has High Energy (100). It’s a valid physical move, but it doesn't help."*  
* **Scenario B (Moving toward the Mug):** JEPA predicts the arm will be near the mug. The EBM says: *"This state matches the Configurator’s goal. Low Energy (10)\! Pursue this path."*

### **3\. Why the Crisps are still "Useful" to JEPA**

Even though the robot doesn't *want* the crisps, JEPA still tracks them as **physical constraints**.

* If the bag of crisps is in the way of the mug, JEPA's internal "world model" knows that the arm cannot pass through the space occupied by the crisps.  
* In latent space, the "Crisps" representation creates a **High-Energy Barrier**.

**The Key Distinction:**

* **JEPA** provides the **Physics**: "The crisps are a solid object; the mug is a solid object."  
* **The Configurator** provides the **Will**: "I want the blue one."

**Takeway: The "Goal-Conditioned" JEPA**

The reason it works in practice is that modern JEPA models are often **Goal-Conditioned**. When we train the robot, we feed the "target image" (a picture of the mug in hand) into a special part of the encoder.

This tells the JEPA Predictor: *"Predict the sequence of latent states that leads to THIS specific target representation."*

