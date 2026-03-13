This taxonomy was created by: [zhuokaiz](https://x.com/zhuokaiz)
## 1) Joint Embedding Predictive Architecture (JEPA)  
**Representatives:** AMI Labs (@ylecun), V-JEPA 2

The core bet is that pixel reconstruction is an inefficient objective for learning the abstractions needed for physical understanding. LeCun has argued this for years: predicting every future pixel is intractable in stochastic environments. JEPA avoids this by predicting in a learned latent space.

Concretely, JEPA trains:
- an **encoder** that maps video patches to representations, and
- a **predictor** that forecasts masked regions in representation space (not pixel space).

That design choice is crucial.

A pixel-generative model must commit to low-level details (exact texture, lighting, leaf position) that are inherently unpredictable. By operating on abstract embeddings, JEPA can capture statements like “the ball will fall off the table” without hallucinating every intermediate frame.

**V-JEPA 2** is the clearest large-scale proof point so far:
- **1.2B parameters**
- pre-trained on **1M+ hours of video**
- self-supervised masked prediction (no labels, no text)

The second stage is where it gets compelling: only **62 hours** of robot data (DROID) can produce an action-conditioned world model for **zero-shot planning**. The robot samples candidate action sequences, rolls them forward in the model, and selects the one whose predicted outcome best matches a goal image—even in unseen objects/environments.

The real headline is **data efficiency**.  
62 hours is tiny. It suggests that broad self-supervised video pretraining can provide enough physical priors that downstream domains need far less task-specific data.

AMI Labs is LeCun’s attempt to push this beyond research, initially in healthcare and robotics—domains where data efficiency and physical reasoning matter. It remains a long-horizon commercialization effort.

---

## 2) Spatial Intelligence (3D World Models)  
**Representative:** World Labs (@drfeifei)

Where JEPA asks, “What happens next?”, this approach asks, “What does the world look like in 3D, and how can we build/manipulate it?”

The thesis: true understanding needs explicit spatial structure—geometry, depth, persistence, and viewpoint consistency—not only temporal prediction.

So this is a different bet from JEPA:
- JEPA emphasizes **abstract dynamics**.
- Spatial intelligence emphasizes **structured 3D scene representations** you can directly operate on.

World Labs’ product **Marble** generates persistent 3D environments from images, text, video, or 3D layouts. “Persistent” is key: unlike linear video generation, outputs are coherent 3D scenes you can orbit, edit, and export as meshes. That positions it closer to a 3D creation system than a pure predictive model.

This builds on neural 3D representation work (e.g., NeRFs, 3D Gaussian Splatting), but shifts toward **generation** rather than reconstruction. The challenge is physical plausibility: stable geometry, sensible occlusion, and consistent lighting in worlds that never existed.

---

## 3) Learned Simulation (Generative Video + Latent-Space RL)  
**Representatives:** Google DeepMind (Genie 3, Dreamer V3/V4), Runway GWM-1

This category merges two lineages that are converging fast:

1. **Generative video models** that simulate interactive worlds  
2. **RL world models** that train policies in imagination

### 3.1 Video-generation lineage
**Genie 3** is the cleanest example: text prompt in, navigable environment out (24 fps, 720p), with multi-minute consistency. It learns interaction dynamics from data rather than relying on explicit hand-built simulators.

Architecturally, generation is autoregressive and conditioned on user actions. Each frame depends on previous frames + current input (move/turn/look), so the model must maintain implicit spatial memory.

Consistency is impressive, but still short of what long-horizon agent training would require.

**Runway GWM-1** uses a similar autoregressive foundation (on Gen-4.5) and splits into:
- Worlds
- Robotics
- Avatars

That split suggests practical generality is still being decomposed by action space and use case.

### 3.2 RL world-model lineage
The **Dreamer** series has deeper history:
- learn latent dynamics from observations,
- roll out imagined trajectories in latent space,
- optimize policy via backprop through model predictions.

No real-environment interaction is needed during policy learning.

Highlights:
- **Dreamer V3:** first AI to get diamonds in Minecraft without human data
- **Dreamer 4:** same outcome purely offline (no environment interaction)

Dreamer 4 also shifted toward a more scalable transformer-style world model and introduced **shortcut forcing**, enabling jumps from noisy to clean predictions in ~4 steps instead of diffusion-like 64-step regimes. This enables real-time inference on a single H100.

These lineages used to feel separate (video generation vs policy learning), but now overlap:
- Humans can play inside Dreamer 4’s world model
- Genie 3 is used for SIMA agent training

The convergence point: both need models that accurately simulate action-conditioned dynamics over long horizons.

Open question (often raised by LeCun): does generating physically plausible pixels imply genuine physics understanding—or just appearance matching? Dreamer 4 is a strong empirical counterpoint, though Minecraft remains a constrained domain.

---

## 4) Physical AI Infrastructure (Simulation Platform)  
**Representative:** NVIDIA Cosmos

NVIDIA’s strategy: don’t only build a world model—build the **platform** everyone uses to build theirs.

**Cosmos** (CES Jan 2025) spans the stack:
- data curation pipeline  
- visual tokenizer (reported 8x better compression vs prior SOTA)  
- training with NeMo  
- deployment via NIM microservices

Claimed scale:
- **20M hours** of video processed in **14 days** on Blackwell (vs 3+ years on CPU)
- foundation models trained on **9,000T tokens** from real-world video (driving, industrial, robotics, human activity)

Model families:
- **Diffusion-based** (continuous latent tokens)
- **Autoregressive transformer-based** (discrete token prediction)

Both are fine-tunable per domain.

Top-level product families:
- **Predict:** forecast future video states from text/image/video inputs  
- **Transfer:** sim-to-real adaptation  
- **Reason (GTC 2025):** chain-of-thought-style physical scene reasoning (spatiotemporal, causal interaction, video QA)

---

## 5) Active Inference  
**Representative:** VERSES AI (Karl Friston)

This is the outlier: rooted in computational neuroscience rather than mainstream deep learning.

Friston’s **Free Energy Principle** frames intelligence as prediction + action to minimize surprise (formally, variational free energy as an upper bound on surprise).

Compared to reward-centric RL, active inference minimizes expected free energy, blending:
- goal-directed preferences
- epistemic value (uncertainty reduction)

That naturally drives exploration into uncertain states.

VERSES’ **AXIOM** architecture differs sharply from neural world models:
- not a single monolithic function approximator
- structured generative model with discrete objects, typed attributes, and relations
- Bayesian inference via message passing (vs gradient descent-only learning)

Practical implications:
- **Interpretable** beliefs (inspectable per object)
- **Compositionality** (add object types without full retraining)
- **High data efficiency**

In robotics, they report hierarchical multi-agent setups:
- each robot joint as a local active-inference agent
- higher-level agents for planning
- coordination via shared hierarchical beliefs

This supports online adaptation in unfamiliar environments without retraining. Their commercial product (**Genius**, April 2025) and AXIOM benchmarks report competitive control performance with substantially lower data requirements than standard RL baselines.

---

## Bottom line

These five categories are less direct competitors than complementary layers:

- **JEPA:** compresses physical understanding  
- **Spatial intelligence:** reconstructs/manipulates 3D structure  
- **Learned simulation:** trains agents via generated experience  
- **NVIDIA infrastructure:** supplies the tooling stack  
- **Active inference:** offers a distinct computational theory of intelligence
