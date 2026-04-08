# HERALD: A Formal Coordination Language for Heterogeneous Robot Fleets

### The Problem

As humanoid robots proliferate across hospitals, warehouses, farms, and public spaces, fleets of machines with differing hardware, sensors, and capabilities must cooperate on shared tasks - moving heavy loads, running assembly lines, managing shared spaces - without centralised control and over bandwidth-constrained wireless links (BLE/LoRa). No existing framework satisfies all requirements simultaneously: coordination languages like Buzz lack long-horizon planning; formal methods like Klaim assume reliable channels; ontology systems like KnowRob are too heavy for constrained links; Behaviour Trees are single-agent; and the Contract Net Protocol has no fault model. When a robot fails Byzantine - continuing to transmit incorrect coordination signals due to sensor failure or software fault - existing systems have no principled response.

---

### The Research Question

> How can a formal, compact, verifiable, and human-interpretable coordination language enable heterogeneous robots to execute cooperative tasks spanning long-horizon co-planned sequences and reactive atomic interactions, over bandwidth-constrained lossy channels, while tolerating Byzantine agent failures?

---

### Proposed Solution: The Robot Coordination Calculus (RCC)

HERALD defines RCC, a coordination framework with four interlocking contributions:

**1. Dual-Mode Coordination.** RCC distinguishes between two coordination regimes. In planning mode, robots jointly construct and negotiate multi-step task graphs for long-horizon operations. In reactive mode, robots exchange atomic, event-triggered signals for sub-second responses. Critically, the transition rules between these two regimes are not hand-coded but are learned from experience, addressing a gap no prior system treats as a first-class concern.

**2. Learned Coordination Semantics.** Rather than prescribing a fixed set of agent states and hand-crafted transition rules, HERALD treats the coordination protocol itself as a learned policy. Each robot learns, through reinforcement learning or imitation of successful multi-robot demonstrations, when to propose a plan, when to defer, when to interrupt, and when to trigger a replanning cycle. This allows the protocol to generalise across task types and robot configurations that were not anticipated at design time, while the formal language layer constrains the message vocabulary so that learned behaviours remain interpretable and verifiable.

**3. Byzantine Fault Tolerance.** Extending classical BFT to mobile wireless fleets, HERALD monitors agent behaviour against Signal Temporal Logic robustness margins. Agents whose behaviour consistently deviates from learned norms trigger suspicion signals; isolation requires a quorum of independent suspicions, producing an auditable isolation decision. The suspicion threshold itself is a learned quantity, calibrated against simulated fault distributions during training rather than set by hand.

**4. Implementable Wire Format.** RCC messages are serialised via Protocol Buffers, achieving a 60 to 80 percent size reduction over JSON and targeting under 512 bytes per exchange. The schema operationalises IEEE 1872-2015 (CORA) for constrained wireless environments, bridging the gap between KnowRob's expressiveness and LoRa's payload budget.

A bidirectional natural language transpiler converts RCC message logs, including plan negotiations and Byzantine events, into human-readable summaries for operator oversight.

---

### Positioning Against Prior Art

| | Long-Horizon | Reactive | Byzantine-Tolerant | Bandwidth-Constrained | Formally Verified |
|---|:---:|:---:|:---:|:---:|:---:|
| Buzz | ✗ | ✓ | ✗ | ✓ | ✗ |
| Klaim | ✓ | ✓ | ✗ | ✗ | ✓ |
| Behaviour Trees | ✗ | ✓ | ✗ | - | Partial |
| Contract Net | Partial | ✗ | ✗ | ✗ | ✗ |
| **HERALD / RCC** | **✓** | **✓** | **✓** | **✓** | **✓** |

---

### Validation

Seven task classes ground empirical evaluation, ranging from synchronous load-carrying (Class A, reactive) through Byzantine-exposed dynamic re-allocation (Class D) to multi-hour warehouse campaigns (Class G, fully co-planned). A Master's thesis covers Classes A through C in simulation; a PhD extends to all classes with physical heterogeneous hardware, a full formal verification model, and a human factors study on operator interpretability.

---

### Expected Contributions

An open RCC specification with a learned coordination semantics layer; a fault-tolerant evolution of the Contract Net Protocol; an STL-grounded Byzantine detection mechanism with learned suspicion thresholds; an open Protocol Buffers schema aligned with IEEE 1872; a ROS2 reference implementation; a community benchmark suite; and a human factors study, collectively advancing the state of the art in multi-robot coordination languages.
