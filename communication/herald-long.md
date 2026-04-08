# Multi-Robot Coordination Language for Heterogeneous Autonomous Systems
## Research Problem Formulation — Revised with Related Work

---

### Problem Statement

As humanoid robots proliferate across public and private environments, a fundamental gap exists in how machines with heterogeneous hardware configurations — differing degrees of freedom, sensor modalities, and computational capabilities — can coordinate complex multi-agent tasks under real-world communication constraints. Existing approaches either rely on centralised orchestration (fragile, single point of failure), assume homogeneous fleets (unrealistic), or borrow human natural language (ambiguous, unverifiable, bandwidth-heavy).

A particularly under-explored dimension is the **temporal horizon of coordination**: some tasks require robots to co-plan sequences of interdependent actions far into the future — a surgical supply chain across a hospital wing, a multi-stage warehouse pick-and-pack operation — while others demand purely reactive, atomic responses to immediate stimuli, such as catching a falling object or yielding to a human in a corridor. A viable coordination language must span this full spectrum without collapsing into either a rigid planning formalism or an ad-hoc reactive protocol.

A further dimension, largely ignored by existing frameworks, is **fault semantics**: in a decentralised fleet operating over lossy wireless channels, a robot may not merely go silent — it may continue transmitting incorrect or contradictory coordination messages due to sensor failure, software fault, or adversarial conditions. The coordination language must therefore incorporate mechanisms for detecting, isolating, and recovering from such Byzantine agents without halting the broader task.

This research asks:

> **How can a formal, compact, verifiable, and human-interpretable inter-robot coordination language be designed to enable asynchronous, interruptible, bandwidth-constrained multi-agent task execution — spanning both long-horizon co-planned sequences and reactive atomic interactions — while remaining resilient to Byzantine agent behaviour, across heterogeneous robotic platforms?**

---

### Research Title (candidates)

- *"HERALD: Heterogeneous Embodied Robot Asynchronous Language for Distributed Coordination"*
- *"Towards a Formal Machine-to-Machine Protocol for Cooperative Task Execution in Heterogeneous Robot Fleets"*
- *"Beyond Natural Language: A Verifiable Coordination Calculus for Multi-Robot Systems Across Planning Horizons"*

---

### Why This is a PhD/Master-Level Problem

This problem sits at the intersection of seven research fields simultaneously, none of which individually resolves it:

| Field | Contribution | Gap |
|---|---|---|
| Multi-Agent Systems | Task allocation, coalition formation | Assumes shared world model, benign agents |
| Robotics & Control | Kinematics, sensor fusion | Not concerned with inter-robot semantics |
| Formal Languages & Verification | Temporal logic, process calculi | Not grounded in embodied physical action |
| AI Planning (PDDL, HTN) | Long-horizon sequential reasoning | Not designed for bandwidth-constrained peer exchange |
| Distributed Systems | Fault tolerance, consensus protocols | Not semantically grounded in physical tasks |
| Wireless Communications | BLE, LoRa, 5G protocols | Agnostic to semantic payload |
| Human-Computer Interaction | Explainability, operator dashboards | Downstream, not foundational |

---

### Related Research & Prior Art

Positioning HERALD requires precise engagement with existing work across four clusters. In each case, the goal is not merely to cite prior art but to identify the exact gap that HERALD fills — the specific design decision or theoretical commitment that the prior work makes which HERALD either extends, refutes, or supersedes.

---

#### A. Coordination Languages & Swarm Programming

**Buzz (Pinciroli & Beltrame, 2016)**
Buzz is the most direct predecessor to HERALD and must be engaged with carefully. It is a dedicated scripting language for heterogeneous robot swarms, designed around the abstraction of *neighbour-to-neighbour* communication — each robot executes the same Buzz script, exchanging local state with physically proximate peers. Its key strength is handling emergent, decentralised behaviours across fleets of robots with differing hardware through a unified virtual machine. However, Buzz has two limitations that motivate HERALD. First, it has no native representation of *long-horizon task graphs* — Buzz scripts are essentially reactive loops over local neighbourhood data, with no mechanism for agents to jointly construct, negotiate, and commit to a multi-step plan that extends beyond the current timestep. Second, Buzz provides no formal fault model — a Byzantine neighbour broadcasting corrupt state is indistinguishable from a correct one. HERALD directly addresses both gaps: its plan-mode primitives introduce long-horizon co-planning absent from Buzz, and its Byzantine detection layer adds fault semantics that Buzz entirely lacks.

**Voltron**
Voltron represents a more recent shift towards *data-driven* coordination, where robots share compressed observations and learned representations rather than symbolic task descriptions. This approach is compelling in perception-heavy settings but introduces a verifiability problem: when coordination is mediated by learned embeddings, it becomes difficult or impossible to formally verify that a coordination sequence satisfies safety or liveness properties. HERALD takes the opposite position — prioritising a symbolic, formally verifiable core — while acknowledging that Voltron-style learned representations could serve as an input layer that feeds into HERALD's capability ontology.

**Klaim (Kernel Language for Agent Interaction and Mobility)**
Klaim is a process calculus designed specifically for mobile agents interacting through distributed tuple spaces. Its treatment of *locality* — agents can read from and write to named locations, including remote ones — makes it a natural formal ancestor for HERALD's co-planning protocol, where agents must read and update shared partial task graphs. Klaim's operational semantics, expressed as a labelled transition system over located processes, directly inform the formal structure of RCC's state machine. The key departure is that Klaim assumes reliable, synchronous communication — an assumption that is untenable over BLE/LoRa channels with packet loss and variable latency. HERALD's contribution relative to Klaim is the introduction of asynchronous, lossy, Byzantine-aware semantics into a Klaim-inspired formal framework.

---

#### B. Formal Verification & Temporal Logic

**LTL Synthesis — Kress-Gazit et al.**
Hadas Kress-Gazit's body of work on *correct-by-construction* robot controllers represents the gold standard for the verifiable requirement in HERALD. Her framework takes structured English specifications — essentially a restricted natural language — and synthesises provably correct robot controllers by compiling the specification into Linear Temporal Logic (LTL) and solving a two-player game between the robot and its environment. This is directly relevant to HERALD's verification layer: the formal correctness guarantees Kress-Gazit establishes for single-robot controllers must be extended to *multi-agent* settings where the "environment" includes other robots that are simultaneously co-planning. The open research question HERALD addresses here is whether LTL synthesis can be lifted to a distributed setting where each agent holds only a partial view of the task graph and must verify its *local* obligations without access to the global plan — a problem Kress-Gazit's framework does not address.

**Signal Temporal Logic (STL)**
Unlike LTL, which reasons about the ordering of events without reference to real time, STL introduces explicit time bounds on temporal constraints — for example, "Robot A must reach position X within 10 seconds of Robot B sending COMMIT." This distinction is critical for HERALD's Class A and Class B tasks, where physical coordination (lifting a load jointly, handing off an item on an assembly line) imposes hard real-time constraints that LTL cannot express. STL also supports *quantitative* satisfaction — a formula is not merely true or false but has a *robustness* score measuring how far the system is from violation. This robustness margin is useful for HERALD's fault layer: an agent whose behaviour has low STL robustness across multiple time windows is a candidate for a `SUSPECT` flag, providing a principled, metric-based criterion for Byzantine detection grounded in temporal logic rather than heuristics.

---

#### C. Capability Ontologies

**KnowRob (Tenorth & Beetz)**
KnowRob is the most comprehensive knowledge processing system for robots, providing a rich OWL-based ontology for representing object properties, actions, robot capabilities, and episodic memory of past actions. It is demonstrably powerful in laboratory settings, enabling robots to reason about tasks at a high semantic level. However, KnowRob's computational and representational footprint makes it incompatible with HERALD's bandwidth constraints: a full KnowRob capability description serialised as OWL/RDF routinely exceeds tens of kilobytes per agent, orders of magnitude beyond the LoRa payload budget. The gap this creates is precisely what HERALD's capability ontology layer fills — what might be termed a *KnowRob-Lite* or *Minimal Semantic Web for Robots*: a capability description format that retains sufficient expressiveness for task negotiation while being serialisable in under 200 bytes. This is not a trivial compression of KnowRob but a principled re-design of what semantic information is *necessary and sufficient* for coordination, as opposed to general robot reasoning.

**IEEE 1872-2015: Standard Ontologies for Robotics and Automation (CORA)**
The IEEE 1872 standard defines a core ontology for robotics and automation (CORA), establishing a shared vocabulary for robot capabilities, motion, and environment interaction. HERALD's capability ontology must align with CORA to ensure academic rigour, cross-platform compatibility, and adoption potential. Specifically, CORA's taxonomy of *capability*, *action*, and *agent* concepts provides the semantic backbone onto which HERALD maps its compact binary encodings. The research contribution is demonstrating that a CORA-aligned ontology can be losslessly projected into a Protocol Buffers schema without sacrificing the standard's expressive coverage for the task classes under study — effectively operationalising IEEE 1872 for constrained wireless environments.

---

#### D. Hybrid Planning & Reactive Architectures

**Behaviour Trees (BTs)**
Behaviour Trees have become the de facto industry standard for robot task execution, adopted in ROS2 Nav2, game AI engines, and autonomous vehicle stacks. Their appeal lies in their natural handling of *interruptibility* — a fallback branch can interrupt a running action when a condition changes — and their composability, allowing complex behaviours to be constructed from simple reusable nodes. BTs are therefore a serious alternative to HERALD that must be explicitly argued against. The key distinction is architectural: a Behaviour Tree is a *single-agent execution framework* — it governs what *one* robot does given its local state. Coordinating multiple robots with BTs requires an external mechanism (typically a centralised planner or shared blackboard) to synchronise trees across agents. HERALD, by contrast, is a *multi-agent communication language* — its primitives are inherently about what agents say to each other, not what they individually execute. A robot running a BT can use HERALD as the inter-agent signalling layer that drives its tree's condition nodes, making the two complementary rather than competing. However, the research must formally demonstrate that the HERALD co-planning protocol can guarantee fleet-level properties (deadlock-freedom, liveness) that a collection of independently executing BTs — without a coordination language — cannot.

**Contract Net Protocol (Smith, 1980)**
The Contract Net Protocol is the foundational ancestor of HERALD's co-planning negotiation mechanism. In CNP, a manager agent broadcasts a task announcement, contractor agents submit bids, and the manager awards the contract to the best bidder. This maps naturally onto HERALD's `PROPOSE / ACCEPT / REJECT / COMMIT` primitives. However, CNP makes three assumptions that HERALD must relax. First, CNP assumes a reliable broadcast channel — HERALD operates over lossy LoRa/BLE where announcements may not reach all agents. Second, CNP is inherently single-round — there is no native mechanism for iterative plan negotiation or plan repair after commitment. Third, CNP has no fault model — a contractor that accepts a contract and then fails Byzantine has no defined handling. HERALD's co-planning protocol is therefore best understood as a *bandwidth-efficient, fault-tolerant, multi-round evolution of CNP*, grounded in the forty years of multi-agent systems research that CNP has generated, but extending it in the three dimensions where CNP is insufficient for the target deployment context.

---

#### Synthesis: The Precise Gap HERALD Fills

The following table summarises the exact dimensions along which each prior work falls short and how HERALD addresses each:

| Prior Work | Long-Horizon Planning | Reactive Primitives | Byzantine Tolerance | Bandwidth-Constrained | Formal Verification |
|---|:---:|:---:|:---:|:---:|:---:|
| Buzz | ✗ | ✓ | ✗ | ✓ | ✗ |
| Voltron | ✗ | ✓ | ✗ | ✓ | ✗ |
| Klaim | ✓ | ✓ | ✗ | ✗ | ✓ |
| BTs | ✗ (single-agent) | ✓ | ✗ | N/A | Partial |
| KnowRob | ✓ (reasoning) | ✗ | ✗ | ✗ | ✗ |
| Contract Net | Partial | ✗ | ✗ | ✗ | ✗ |
| LTL Synthesis | ✓ (single-agent) | ✗ | ✗ | N/A | ✓ |
| **HERALD / RCC** | **✓** | **✓** | **✓** | **✓** | **✓** |

No single existing system satisfies all five properties simultaneously. HERALD is the first framework designed to do so, and each column of this table corresponds to a research contribution.

---

### Research Objectives

**Primary Objective**
Design and evaluate a formal coordination language — the *Robot Coordination Calculus (RCC)* — that allows heterogeneous robots to negotiate shared task representations and execute cooperative work under constrained, lossy, asynchronous, and potentially faulty communication environments, across the full spectrum from deliberative long-horizon co-planning to reactive atomic action.

**Secondary Objectives**

1. Define a minimal **capability ontology** aligned with IEEE 1872-2015/CORA — a compact, binary-serialisable vocabulary for robots to advertise capabilities without exposing full internal state, filling the gap between KnowRob's expressiveness and LoRa's bandwidth budget
2. Design a **dual-mode task representation** distinguishing plan-mode from react-mode, with formally specified transitions constituting an extension of Klaim's mobility semantics to asynchronous lossy channels
3. Define the **operational semantics of RCC** — complete labelled transition systems for all agent coordination states, model-checked in TLA⁺, extending Kress-Gazit's single-agent LTL synthesis to distributed partial-view settings
4. Develop a **co-planning protocol** — a fault-tolerant, multi-round evolution of the Contract Net Protocol supporting iterative plan negotiation, versioned plan graphs, and surgical repair under agent failure
5. Develop a **Byzantine fault detection and isolation mechanism** — using STL robustness margins as a principled, metric-based criterion for generating `SUSPECT` messages, with quorum-based isolation extending classical PBFT to mobile wireless fleets
6. Develop a **formal verification layer** proving coordination sequences are deadlock-free, liveness-preserving, and safe, expressed in STL for time-bound constraints (Class A, B) and LTL for ordering properties (Class C–G)
7. Create a **human-readable transpiler** converting compact RCC/protobuf message logs into natural language summaries, including Byzantine suspect events and plan repair sequences, for operator oversight
8. Validate across **canonical task classes** in simulation and on physical hardware

---

### The Planning Horizon Problem

One of the core theoretical contributions of this work is the explicit treatment of **coordination across temporal scales** — a dimension largely absent from existing multi-robot communication frameworks.

**Long-Horizon Co-Planning** tasks require robots to jointly construct a shared, temporally ordered graph of interdependent actions before or during execution. These plans may span minutes to hours, involve conditional branches, resource reservations, and role assignments that must be agreed upon in advance. The coordination language must support compact serialisation of partial task graphs exchangeable over low-bandwidth links, distributed plan negotiation, plan versioning and consistency, and graceful replanning when a step fails or a robot drops out.

**Reactive Atomic Tasks** require sub-second coordination with minimal message overhead. Here, the language must support event-driven primitives (ALERT, YIELD, CATCH, HOLD) that carry maximal meaning in minimal bytes, interruption of in-progress plan execution without corrupting shared plan state, and stateless interpretability — a robot that missed prior context can still act correctly on a reactive message alone.

**The Transition Problem** between these two modes is itself a novel research question that neither Behaviour Trees nor Contract Net addresses. When does a reactive event warrant a full plan revision? How is that decision negotiated across agents? How is the existing long-horizon plan suspended, patched, or abandoned? These are first-class design concerns in HERALD.

---

### Operational Semantics of RCC

A central PhD-level contribution is the rigorous definition of RCC's **operational semantics** — the formal rules governing how agents change state in response to messages, time, and environmental events. Without this, RCC remains an informal sketch rather than a verifiable system.

Each agent is modelled as a **labelled transition system** over a finite set of coordination states:

```
IDLE | PROPOSING | NEGOTIATING | COMMITTED | EXECUTING | SUSPENDED | FAULTED | ISOLATED
```

**Core Message Primitives**

```
PROPOSE(task_id, role, deadline, priority)
ACCEPT(task_id, role)
REJECT(task_id, reason)       -- reason ∈ {BUSY, INCAPABLE, TIMEOUT}
COMMIT(task_id, plan_hash)
EXECUTE(task_id, step_id)
PAUSE(task_id, reason)
RESUME(task_id, step_id)
ABORT(task_id, reason)
REPLAN(task_id, new_graph)
HEARTBEAT(agent_id, state, timestamp)
SUSPECT(agent_id, evidence)
ISOLATE(agent_id, quorum_signature)
```

**Representative State Transitions**

*Scenario 1: Proposal under busy agent — plan-mode*

When agent A sends `PROPOSE(Task_G, Role_1)` to agent B currently in state `EXECUTING`:

```
B.state = EXECUTING
B receives PROPOSE(Task_G, Role_1, deadline=T, priority=P)

if P > current_task.priority:
    B → SUSPENDED (current task checkpointed)
    B sends ACCEPT(Task_G, Role_1)
    B → NEGOTIATING
else if deadline(T) > estimated_completion(current_task):
    B sends REJECT(Task_G, BUSY)
    A starts timeout counter τ
    if τ expires without alternate ACCEPT:
        A → REPLAN or ABORT depending on task_class
else:
    B queues PROPOSE internally
    B sends DEFER(Task_G, eta=estimated_completion)
    A → NEGOTIATING (waiting state, non-blocking)
```

*Scenario 2: Reactive interrupt during plan execution*

```
A.state = EXECUTING(Task_G, step=k)
A receives ALERT(collision_risk, severity=HIGH)

A → SUSPENDED (step k checkpointed)
A broadcasts PAUSE(Task_G, reason=ALERT)
All COMMITTED agents: EXECUTING → SUSPENDED

if alert resolves within τ_reactive:
    A broadcasts RESUME(Task_G, step=k)
    All agents: SUSPENDED → EXECUTING
else:
    A initiates REPLAN(Task_G, partial_graph_from_step_k)
    All agents: SUSPENDED → NEGOTIATING
```

These transition rules, fully specified for all message types and state combinations, constitute the formal operational semantics of RCC and are the basis for model checking in TLA⁺ or SPIN.

---

### Serialisation: From Theory to Wire

**CBOR (RFC 8949)** achieves 40–60% size reduction over JSON with no schema required, supports rich types, and has native implementations across embedded platforms. A CBOR-encoded `PROPOSE` message with a 10-step partial task graph is estimated at 200–350 bytes — within LoRa's payload budget.

**Protocol Buffers** achieves 60–80% reduction over JSON by replacing field names with integer tags defined in a `.proto` schema. The schema acts as a machine-readable contract between agents and doubles as a formal interface definition, directly aligning with RCC's verifiability goal — the `.proto` file *is* a fragment of the language specification, and its release as an open artefact operationalises IEEE 1872 for constrained wireless environments.

**Proposed approach:** Protocol Buffers for the stable RCC message core, with CBOR as a fallback for schema-less dynamic extensions such as capability ontology advertisements that evolve at runtime. The information-theoretic lower bound on RCC message size — given the expressiveness required by the operational semantics — is itself a formal research question, connecting language design to Shannon's source coding theorem and grounding the serialisation choice in theory rather than engineering convenience alone.

---

### Byzantine Fault Tolerance in Multi-Robot Coordination

Classical multi-robot fault models assume agents either function correctly or go silent. Real deployments violate this: a LIDAR unit returns phantom obstacles, a gyroscope drifts, a software fault causes an agent to broadcast stale plan states. These are Byzantine failures — agents that continue to participate while producing incorrect information — and they are particularly dangerous in cooperative physical tasks.

**Detection Mechanisms**

*Consistency cross-checking.* Each agent maintains a shadow copy of the agreed plan graph. A received message whose embedded `plan_hash` mismatches the local copy, unexplained by a known `REPLAN`, triggers a `SUSPECT` broadcast.

*STL-based anomaly detection.* Agent behaviour is monitored against STL specifications. An agent whose behavioural signals yield consistently low STL robustness scores across multiple time windows — for instance, claiming to be `EXECUTING` step k while its heartbeat timestamps indicate it has been stationary — is flagged as suspect on a principled, metric-based criterion rather than a heuristic threshold.

*Quorum-based isolation.* Isolation requires f+1 independent `SUSPECT` signals from distinct agents, following classical BFT theory adapted for mobile wireless fleets. The `ISOLATE` message carries a `quorum_signature` making the decision auditable. An isolated agent transitions to `FAULTED` and is excluded from further negotiation pending human operator review or a formal re-admission protocol.

**Open Research Questions**
What constitutes admissible *evidence* in a `SUSPECT` message — sensor readings, message logs, physical observations? How should the isolation threshold f be set dynamically as fleet size changes? Can a previously isolated agent re-integrate, and what re-admission protocol ensures it is not still Byzantine? These constitute a standalone research contribution within the broader thesis.

---

### Task Taxonomy

| Class | Description | Horizon | Byzantine Risk |
|---|---|---|---|
| A | Synchronous physical coupling (pallets, beds) | Reactive | Low |
| B | Sequential pipeline / assembly line | Long-horizon co-planned | Medium |
| C | Concurrent spatial tasks (ward sanitisation) | Mixed | Medium |
| D | Dynamic re-allocation with Byzantine exposure | Plan-repair + reactive | High |
| E | Human-in-the-loop handoff | Reactive trigger, planned recovery | Medium |
| F | Environmental negotiation (corridors, doors) | Reactive | High |
| G | Long-horizon campaign (harvest, warehouse) | Fully co-planned | Diffuse/cumulative |

---

### Hypotheses

**H1:** A CORA-aligned capability ontology serialised via Protocol Buffers can represent all task classes, including partial task graph transfers, within 512 bytes per coordination exchange over LoRa/BLE.

**H2:** A TLA⁺ model-checked operational semantics guarantees deadlock-freedom and task liveness before physical execution, reducing task failure rates versus unverified baselines.

**H3:** The dual-mode architecture outperforms single-mode approaches on mixed-horizon tasks, measured by completion rate and replanning latency following agent failure.

**H4:** The STL-grounded Byzantine detection mechanism correctly identifies and isolates faulty agents within a bounded number of message rounds, with zero false positives under tested fault models.

**H5:** Human operators correctly interpret transpiled coordination logs — including Byzantine suspect events and plan repair sequences — with high accuracy within 30 seconds.

**H6:** HERALD remains functional under 30% message loss and 2-second latency characteristic of real BLE/LoRa deployments, degrading gracefully rather than catastrophically.

---

### Differentiation from Existing Work

| Existing Approach | Limitation | This Work |
|---|---|---|
| Buzz | Reactive only, no long-horizon planning, no fault model | Plan-mode + react-mode + Byzantine layer |
| Voltron | Learned representations, unverifiable | Symbolic formal core, verifiable |
| Klaim | Synchronous reliable channels assumed | Asynchronous lossy Byzantine-aware semantics |
| BTs | Single-agent execution, no inter-agent language | Multi-agent coordination language; BTs remain as execution layer |
| KnowRob | Computationally heavy, bandwidth-incompatible | CORA-aligned KnowRob-Lite under 200 bytes |
| Contract Net | Single-round, reliable channel, no fault model | Multi-round, lossy, Byzantine-tolerant evolution |
| LTL Synthesis (Kress-Gazit) | Single-agent, full world model assumed | Distributed partial-view multi-agent extension |
| STL | Specification tool only | Used as Byzantine detection metric (robustness margin) |
| Classical BFT (PBFT) | Stationary nodes, authenticated channels | Adapted for mobile lossy wireless, task-semantically grounded |

---

### Expected Contributions

1. **RCC specification** — formal grammar, operational semantics, dual-mode horizon primitives, and Byzantine fault messages, suitable as an open standard
2. **TLA⁺ model** — complete verification of deadlock-freedom and liveness for all agent state transitions and message types
3. **Distributed co-planning protocol** — fault-tolerant multi-round evolution of Contract Net for partial-state, bandwidth-constrained settings
4. **STL-grounded Byzantine detection protocol** — novel adaptation of BFT consensus to mobile robot fleets with formal bounds on detection latency
5. **Protocol Buffers wire format** — open `.proto` schema operationalising IEEE 1872-2015/CORA for constrained wireless environments
6. **Open-source ROS2 reference implementation** deployable over BLE/LoRa/WiFi
7. **Benchmark suite** across the full task taxonomy, released as a community resource
8. **Human factors study** on operator interpretability across planning, reactive, and fault modes

---

### Scope Calibration

| Track | Scope |
|---|---|
| **Master's thesis** | Classes A, B, C; crash-stop faults only; simulation only; RCC design + TLA⁺ verification for plan-mode and react-mode; CORA-aligned ontology; protobuf schema; transpiler prototype |
| **PhD thesis** | All classes A–G; full STL-grounded Byzantine detection (Classes D, F); simulation + physical validation on heterogeneous hardware; complete TLA⁺ model; human-readability user study; RCC and protobuf schema submitted as open standard |
