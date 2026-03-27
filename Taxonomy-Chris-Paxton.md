# Taxonomy of World Models (Chris Paxton)

> **Source:** [It Can Think (Substack)](https://itcanthink.substack.com/p/will-world-models-allow-robots-to)
> **Author:** Chris Paxton

## Introduction
While AI in general is advancing rapidly, intelligence in robotics still faces significant deployment hurdles. However, the recent explosion in video generation capabilities (e.g., Google’s Veo 3, ByteDance’s Seedance 2.0) has fueled a surge in **World Models**—video-predictive models for robot tasks. These models aim to close the "robot data gap" by leveraging massive amounts of internet video data to learn priors about physical interaction.

## Core Taxonomy of World Models
Paxton identifies three primary ways to set up the world modeling problem, distinguished by how they handle actions and state transitions.

### 1. Action-Conditioned World Models
These are the "classic" formulation, predicting future states based on current state and a specific action vector.
- **Formulation:** $x' = f(x, a)$
- **Key Characteristics:** 
  - Theoretically justified; the "model" in model-based reinforcement learning.
  - Championed by Yann LeCun (JEPA-style).
  - **Examples:** V-JEPA 2 [1], Dreamer v4 [2], RISE [3].
- **Pros/Cons:** Purest form but suffers from rapid error accumulation and requires explicit action labels (which internet video lacks).

### 2. Video World Models (with Inverse Dynamics)
These models generate video first and then recover actions using a separate "inverse dynamics model."
- **Formulation:** $x' = f(x)$ and $a = g(x, x')$
- **Key Characteristics:**
  - Can leverage massive-scale video data without action labels.
  - Recovers actions from a model $g$ trained on smaller labeled datasets.
  - **Examples:** NVIDIA DreamGen [4], 1x World Model, LingBot-VA [5].
- **Pros/Cons:** Best for leveraging internet data; however, they require "working backwards" to find the control signals.

### 3. Joint World Action Models (WAMs)
Policies that predict both video sequences and actions jointly.
- **Formulation:** $x', a = f(x)$
- **Key Characteristics:**
  - Learns how the world evolves and what the robot is doing simultaneously.
  - Does not necessarily need to decode video at test time (can be latent-only).
  - **Examples:** DreamZero [6], Fast WAM [7].
- **Pros/Cons:** Appear to generalize better than standard VLAs (Vision-Language-Actions) by implicitly learning world evolution.

---

## 2D vs. 3D World Models
Separately from the action logic, there is a distinction in representation:
- **2D Generative:** Predicting pixel-space or latent-space video.
- **3D Generative:** Creating a 3D scene representation (e.g., **PointWorld** [9] or World Labs' **Marble**).

## Why This Matters: The History of Planning
Planning for robots has historically relied on human-defined dynamics models to solve symbol grounding (e.g., what does "grasp by the handle" actually mean in physical space?). 
- **The Gap:** While LLMs are good at high-level reasoning, they struggle with "grounding" those symbols in the physical world.
- **The Solution:** World models provide the dynamics necessary to convert symbolic instructions into measurable real-world values using the "near infinite" data available in video.

## Hybrid & Emerging Approaches
- **DualWorld:** Combines high-level video generation (WAN 2.2) for long-horizon planning with a local JEPA model for short-horizon control. This allows for injecting tactile/force information into the local controller while benefiting from the generalization of the large video model.
- **System 1 / System 2 Reasoning:** A potential path where video models handle the "intuitive" world evolution while local dynamics models handle "precision" tracking.

## Current Industry Pulse
- **Investment:** Massive capital is flowing into world models (e.g., $1.03B for LeCun’s Advanced Machine Intelligence, $28M for Moonlake AI).
- **Reality Check:** Historically, world models have underperformed. However, the field is shifting due to better video generation backbones and the availability of massive training data.
- **Reliability:** Currently, these models do not yet match the high reliability of on-robot RL (like RL-100 or pi-0.6), but are considered the most promising path forward for generalization.

## References
1. [1] **V-JEPA 2** (Assran et al., 2025): Self-Supervised Video Models for Planning.
2. [2] **DreamerV4** (Hafner et al., 2025): Training Agents Inside Scalable World Models.
3. [3] **RISE** (Yang et al., 2026): Self-Improving Robot Policy with Compositional World Model.
4. [4] **DreamGen** (Jang et al., 2025): Neural Trajectories for Generalization (NVIDIA GEAR Lab).
5. [5] **LingBot-VA** (Li et al., 2026): Causal World Modeling for Robot Control.
6. [6] **DreamZero** (Ye et al., 2026): WAMs as Zero-shot Policies.
7. [7] **Fast-WAM** (Yuan et al., 2026): Efficiency in World Action Models.
8. [8] **WAM Robustness Study** (Zhang et al., 2026): Comparing WAMs vs. VLAs.
9. [9] **PointWorld** (Chao et al., 2026): Scaling 3D World Models.
10. [10] **STRIPS** (Fikes & Nilsson, 1971): Classical planning foundations.
11. [11] **Interactive World Simulator** (Wang et al., 2025): Online demos for robot policy evaluation.
