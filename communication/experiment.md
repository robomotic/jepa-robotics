# Research Draft: Emergent Behavioral Coordination in Heterogeneous Multi-Agent Systems

## Abstract
This research draft outlines an experimental framework for investigating how two heterogeneous robotic agents can resolve a spatial deadlock through emergent, grounded communication. Unlike traditional approaches that rely on shared geometric maps or pre-defined semantics, this experiment utilizes Behavioral Symbol Grounding and the OaK (Options and Knowledge) architecture to enable agents with incompatible internal representations to build a shared predictive model of their interaction.

---

## Problem Statement
Modern autonomous systems typically rely on **SLAM (Simultaneous Localization and Mapping)** for geometric awareness and **VLA (Vision-Language-Action)** models for high-level reasoning and command execution. These standard modules enable sophisticated navigation and task operation in known or shared environments. However, in scenarios involving heterogeneous agents from different manufacturers, these modules often lack a common foundation. A VLA model trained on one robot's visual latent space cannot inherently interpret the embeddings of another's LIDAR-based system, and SLAM-generated maps may use different coordinate frames or feature representations, making direct geometric data exchange impossible.

The primary scientific constraints include:
*   Incompatible internal world models (high-dimensional embeddings) that cannot be directly compared.
*   Unreliable communication channels where identification must be inferred from behavior rather than static identifiers.
*   A complete lack of shared vocabulary or prior coordination history.
*   A communication protocol restricted to arbitrary binary symbols with no pre-assigned meaning.

The experiment is set in a controlled spatial environment consisting of two 3m × 3m rooms connected by a single 3m long corridor. The corridor is constrained to a width of 0.6m—sufficient for a single agent to pass but too narrow for simultaneous passage. The trial begins with the agents positioned in opposite rooms, requiring a negotiated agreement on passage order to successfully exchange locations.

---

## Theoretical Framework

### Behavioral Symbol Grounding
Symbols in this system acquire meaning through sensorimotor correlation. One agent demonstrates an action while broadcasting a symbol, while the observer measures the effect on its own sensors. This approach bypasses geometric dependencies by focusing on speaker consistency and listener learning to minimize prediction surprise.

### Mutual Information as Reward
The learning signal is driven by **Information Gain (IG)**, derived from reducing entropy in the observer's belief over a symbol's effect. The reward for a demonstration is defined as the change in differential entropy:

$$R = H_{before}(s) - H_{after}(s) = KL( P_{posterior} || P_{prior} )$$

This reward is always non-negative and creates a mutual predictability loop. As the speaker becomes more consistent and the listener's internal world model updates to minimize prediction surprise, the agents converge on a shared understanding.

### Joint-Embedding Predictive Architecture (JEPA)
To manage the high-dimensional complexity of their environments, the robots utilize a **Joint-Embedding Predictive Architecture (JEPA)**. Rather than attempting to reconstruct detailed sensory data (such as pixels or raw range-scans), the agents learn to predict the future state of the environment in an abstract latent space. This approach allows the robots to build a "world model" that focuses on relevant physical dynamics while ignoring non-predictable noise.

Crucially, these models are **conditioned on symbol exchange**. The agents treat the arbitrary symbols as high-level latent variables that resolve uncertainty in the joint embedding space. When a peer robot broadcasts a symbol, the observer's JEPA model uses that symbol as a conditional input to its state-transition predictor:

$$P(\text{next\_latent} \mid \text{current\_latent}, \text{own\_action}, \text{peer\_symbol})$$

By training these conditional predictors, the robots effectively learn the "semantics" of the symbols as modifiers of their world model's dynamics. A grounded symbol becomes a prompt that allows the JEPA model to accurately forecast the peer's movement, even through an opaque, incompatible embedding.

### OaK Architecture
The system employs the **Options and Knowledge (OaK)** framework to manage the task. Negotiation is modeled as a hierarchy of skills (Options), where each skill accumulates a return based on its internal policy and bootstraps into the next phase:

$$G^o_t = \sum_{j=1}^{\tau} \gamma^{j-1} \cdot c_{t+j} + \gamma^\tau \cdot V(S_{t+\tau})$$

Specifically, the experiment defines four primary **Options**:
*   **Identify Peer**: A behavioral handshake skill that initiates when a physical obstruction is detected and terminates when motion correlation confirms the identity of the communicating peer.
*   **Ground Symbols**: A motor babbling and observation skill focused on maximizing the Information Gain cumulant, terminating when mutual understanding falls below a designated entropy threshold.
*   **Negotiate Plan**: A collaborative skill involving the proposal and evaluation of grounded symbol sequences, aiming to maximize a shared social value function.
*   **Execute Plan**: A goal-directed skill that terminates upon reaching the target room or detecting a critical divergence from the agreed-upon path.

Predictive knowledge is represented as **General Value Functions (GVFs)** that continuously answer specific questions about the interaction. These include the **peer effect predictor** (predicting sensor deltas given a symbol), the **hallway clearance forecast** (predicting worst-case spatial constraints during passage), the **goal reachability** metric (predicting expected time-to-goal), and the **plan divergence detector** (detecting when physical reality deviates from the negotiated model).

---

## Protocol Flow and Experimental Mechanics
The experiment investigates the protocol flow through a series of coordinated behavioral phases:

```mermaid
sequenceDiagram
    participant A as Robot A
    participant B as Robot B
    
    rect rgb(240, 240, 240)
    Note over A,B: Identification (Handshake)
    A->>B: Signals intent to perform observable motion
    A-->>A: Performs physical "wiggle"
    B->>B: Correlates detected motion with signal source
    end

    rect rgb(230, 245, 230)
    Note over A,B: Grounding (Demonstration)
    A->>B: Broadcasts arbitrary symbol + Demonstrates action
    B->>B: Measures Information Gain on sensor effect
    B->>A: Reports predictability metrics
    end

    rect rgb(230, 230, 245)
    Note over A,B: Plan Negotiation
    A->>B: Proposes sequence of grounded symbols
    B->>B: Simulates cumulative effect using internal model
    B->>A: Accepts or Counters Proposal
    end

    rect rgb(245, 230, 230)
    Note over A,B: Execution & Monitoring
    A & B: Execute agreed behavioral sequence
    A & B: Monitor internal predictions for divergence
    end
```

---

## Evaluation Metrics
The success of the proposed framework is evaluated along both axes of communication and physical progress, applying the following metrics:

*   **Mutual Understanding Metric ($U(t)$)**: The mean entropy across the entire symbol vocabulary. Coordination is achieved as $U(t)$ converges toward zero.
    $$U(t) = \frac{1}{|V|} \sum_{s \in V} H(P(\Delta | s))$$

*   **Grounded Task Efficiency (GTE)**: A composite metric directly linking representational understanding to physical task performance. It is defined as the total accumulated task progress (e.g., room swapping distance) scaled by the inverse of the cumulative communicative uncertainty. This metric penalizes agents that eventually succeed but maintain high entropy or require excessive demonstration:
    $$\text{GTE} = \frac{\sum R_{task}}{\int U(t) dt}$$

*   **Predictive Accuracy**: The statistical correlation between an agent's JEPA-predicted sensor deltas for a proposed symbol sequence and the actual sensory divergence observed during execution.

*   **Negotiation Efficiency**: The temporal cost (elapsed time or communication rounds) from the initial handshake phase to a mutually accepted action plan.

*   **Goal Success Rate**: The percentage of trials where both agents successfully resolve the hallway deadlock and achieve the room-swap without physical collision.

---

## Expected Outcome
This experiment seeks to prove that complex coordination between black-box robotic systems does not require pre-aligned maps. By grounding arbitrary communication in physical behavior and rewarding mutual predictability, robust cooperation can be achieved in decentralized environments. The intended result is a demonstration that OaK-based agents can autonomously resolve spatial deadlocks through self-driven predictive modeling rather than shared geometric foundations.
