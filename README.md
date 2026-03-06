# SurgiTwin: AI-Powered Digital Twin for Robotic Surgery

![Uploading ChatGPT Image Mar 5, 2026, 04_28_29 PM.png…]()


[![IsaacSim](https://img.shields.io/badge/IsaacSim-5.0.0-silver.svg)](https://docs.isaacsim.omniverse.nvidia.com/5.0.0/index.html)
[![Cosmos](https://img.shields.io/badge/NVIDIA-Cosmos_Reason2--2B-76B900.svg)](https://nvidia-cosmos.github.io/cosmos-cookbook/)
[![React](https://img.shields.io/badge/React-Vite-61DAFB.svg)](https://vitejs.dev/)
[![Python](https://img.shields.io/badge/python-3.11-blue.svg)](https://docs.python.org/3/whatsnew/3.11.html)

---

## Table of Contents

- [Goal](#goal)
- [Approach](#approach)
- [System Architecture](#system-architecture)
  - [NVIDIA Isaac Sim (Simulation Layer)](#nvidia-isaac-sim-simulation-layer)
  - [NVIDIA Cosmos Reason2 (AI Reasoning Layer)](#nvidia-cosmos-reason2-ai-reasoning-layer)
  - [Vision and Sensors (Perception Layer)](#vision-and-sensors-perception-layer)
  - [React and Vite Frontend (Interface Layer)](#react-and-vite-frontend-interface-layer)
  - [FastAPI Orchestrator (Integration Hub)](#fastapi-orchestrator-integration-hub)
- [Data Flow](#data-flow)
- [Impact](#impact)
- [Getting Started](#getting-started)
  - [Prerequisites](#prerequisites)
  - [Startup (4 Terminals)](#startup-4-terminals)
  - [Diagnostics](#diagnostics)
- [Key Components](#key-components)
- [Future Implications](#future-implications)
- [License](#license)

---

## Goal

Build a real-time **digital twin** for robotic surgery that pairs high-fidelity physics simulation with vision-language AI reasoning. SurgiTwin enables a surgeon or researcher to:

1. **Observe** a photorealistic surgical scene rendered by NVIDIA Isaac Sim.
2. **Query** an AI agent (NVIDIA Cosmos Reason2) about spatial relationships, physical feasibility, and next-best actions — all from the robot's egocentric perspective.
3. **Command** the robotic arm through natural language, with Cosmos translating intent into precise IK-space actions that the simulation executes in real time.

The result is a closed-loop system where perception, reasoning, and actuation converge in a single interactive interface.

---

## Approach

SurgiTwin connects four layers through a central FastAPI orchestrator:

| Layer | Technology | Role |
|-------|-----------|------|
| **Simulation** | NVIDIA Isaac Sim + Isaac Lab | GPU-accelerated physics, dVRK PSM robot, photorealistic rendering |
| **Reasoning** | NVIDIA Cosmos Reason2-2B (vLLM) | Egocentric scene understanding, surgical planning, action generation |
| **Perception** | Replicator annotators + WebSocket | Real-time MJPEG frame streaming, robot/needle state extraction |
| **Interface** | React + Vite + Tailwind | Live surgery feed, conversational AI chat, scene state overlays |

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────────────┐
│                        SurgiTwin System                            │
│                                                                     │
│  ┌──────────────┐    ┌──────────────────┐    ┌──────────────────┐  │
│  │  NVIDIA       │    │  NVIDIA Cosmos    │    │  React + Vite    │  │
│  │  Isaac Sim    │───>│  Reason2-2B       │───>│  Frontend        │  │
│  │              │    │                   │    │                   │  │
│  │  - Physics   │    │  - Egocentric     │    │  - Live Feed     │  │
│  │  - dVRK PSM  │    │    Reasoning      │    │  - AI Chat       │  │
│  │  - Rendering │    │  - Physical       │    │  - State Overlay │  │
│  │  - State     │    │    Reasoning      │    │  - Quick Actions │  │
│  │    Machines  │    │  - Action Planning│    │                   │  │
│  └──────┬───────┘    └────────┬─────────┘    └────────┬──────────┘  │
│         │                     │                        │             │
│         └─────────────────────┼────────────────────────┘             │
│                               │                                      │
│                    ┌──────────┴──────────┐                           │
│                    │  FastAPI             │                           │
│                    │  Orchestrator        │                           │
│                    │  (Port 7000)         │                           │
│                    │                      │                           │
│                    │  - WebSocket Bridge  │                           │
│                    │  - Frame Buffer      │                           │
│                    │  - MJPEG Stream      │                           │
│                    │  - REST API          │                           │
│                    └─────────────────────┘                           │
└─────────────────────────────────────────────────────────────────────┘
```

### NVIDIA Isaac Sim (Simulation Layer)

Isaac Sim provides a GPU-accelerated physics simulation environment for the surgical scene:

- **Robot**: da Vinci Research Kit (dVRK) Patient Side Manipulator (PSM) in IK-Absolute mode
- **Scene**: Operating room with suture needle, anatomical organs (bladder, prostate, liver)
- **State Machine**: Autonomous pick-and-lift sequence (REST → APPROACH_ABOVE → APPROACH → GRASP → LIFT) implemented with Warp GPU kernels
- **Action Space**: 8-dimensional IK-Absolute — `[pos_x, pos_y, pos_z, qw, qx, qy, qz, gripper]`
- **Gripper Convention**: `1.0` = open, `-1.0` = closed
- **Command Override**: When Cosmos issues a command, the autonomous state machine yields control for ~100 simulation steps, then resumes

### NVIDIA Cosmos Reason2 (AI Reasoning Layer)

Cosmos Reason2-2B is a vision-language model served via vLLM with 4-bit bitsandbytes quantization:

- **Egocentric Reasoning**: Analyzes the scene from the robot's perspective — "I see a needle 2cm below my gripper"
- **Physical Reasoning**: Evaluates forces, trajectories, collision risks, and grasp feasibility
- **Scene Description**: Generates structured natural-language summaries of the surgical field
- **Action Planning**: Translates high-level instructions ("pick up the needle") into multi-step surgical plans with target poses
- **Input**: JPEG frames from the simulation + robot/needle state metadata
- **Output**: Structured reasoning text and/or `set_target_pose` / `move_delta` commands

### Vision and Sensors (Perception Layer)

The perception pipeline bridges simulation and reasoning:

- **Frame Capture**: Replicator annotators capture viewport images as JPEG at simulation rate
- **WebSocket Protocol**: Binary format — `[4 bytes uint32 LE header_len][JSON header][JPEG payload]`
- **Scene State Extraction**: End-effector position, needle position, gripper state, EE-to-needle distance
- **MJPEG Streaming**: Orchestrator serves `/stream.mjpg` to the browser for real-time video display

### React and Vite Frontend (Interface Layer)

The web interface provides interactive access to the digital twin:

- **Live Surgery Feed**: MJPEG stream with real-time state overlays (EE position, needle position, gripper state)
- **Cosmos Agent Chat**: Conversational interface for querying scene state and issuing commands
- **Aceternity UI Components**: Glowing borders, spotlight hover effects, animated gradients
- **Quick Suggestions**: Pre-built queries — "What should the robot do next?", "Is this approach safe?", "Where is the needle?"
- **Re-describe Button**: Triggers fresh scene analysis at any point during the conversation

### FastAPI Orchestrator (Integration Hub)

The orchestrator is the central nervous system connecting all layers:

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/health` | GET | System status (Isaac connection, Cosmos availability) |
| `/stream.mjpg` | GET | MJPEG live video stream |
| `/scene` | GET | Current robot/needle state JSON |
| `/describe` | POST | Cosmos scene description with current frame |
| `/reason/egocentric` | POST | Custom egocentric reasoning query |
| `/reason/physical` | POST | Custom physical reasoning query |
| `/instruction` | POST | Natural language → robot action |
| `/command/cartesian` | POST | Direct Cartesian command to robot |
| `/plan` | POST | Multi-step surgical plan generation |
| `/plan/status` | GET | Current plan execution state |
| `/plan/step` | POST | Execute specific plan step |
| `/ws/frames` | WebSocket | Isaac Sim → Orchestrator frame stream |
| `/ws/commands` | WebSocket | Orchestrator → Isaac Sim command stream |

---

## Data Flow

```
Isaac Sim                  Orchestrator                 Frontend
   │                           │                           │
   │──── JPEG + state ────────>│                           │
   │     (WebSocket)           │──── MJPEG stream ────────>│
   │                           │                           │
   │                           │<──── "Describe Scene" ────│
   │                           │                           │
   │                           │──── frame + state ──> Cosmos (vLLM)
   │                           │<──── reasoning text ──────│
   │                           │                           │
   │                           │──── AI response ─────────>│
   │                           │                           │
   │                           │<──── "Pick up needle" ────│
   │                           │──── prompt ──────────> Cosmos
   │                           │<──── target_pose ─────────│
   │                           │                           │
   │<──── set_target_pose ─────│                           │
   │      (WebSocket)          │                           │
   │                           │                           │
   │  [Robot moves to target]  │                           │
   │                           │                           │
```

---

## Impact

- **Accelerated Surgical AI Development**: Researchers can iterate on autonomous surgical behaviors without physical hardware, reducing development cycles from weeks to hours.
- **Safe Training Environment**: AI models learn needle manipulation, tissue interaction, and instrument control in simulation before any real-world deployment.
- **Natural Language Robot Control**: Surgeons can communicate intent in plain language rather than programming joint trajectories, lowering the barrier to human-robot collaboration.
- **Real-Time Scene Understanding**: Egocentric reasoning provides spatial awareness that mirrors how a surgeon perceives the operative field, enabling context-aware assistance.
- **Scalable Validation**: GPU-parallelized simulation enables thousands of concurrent surgical scenarios for robust policy validation.

---

## Getting Started

### Prerequisites

- **GPU**: NVIDIA RTX GPU with 12+ GB VRAM (tested on RTX 5070 Ti Laptop)
- **NVIDIA Isaac Sim 5.0.0** with Isaac Lab
- **Conda environment**: `robotic_surgery` (for Isaac Sim scripts)
- **Python 3.11**, **Node.js 18+**
- **vLLM** with bitsandbytes support

### Startup (4 Terminals)

**Terminal 1 — vLLM (Cosmos Reason2, 2-bit quantized)**
```bash
vllm serve nvidia/Cosmos-Reason2-2B \
  --quantization bitsandbytes --load-format bitsandbytes \
  --max-model-len 1024 --gpu-memory-utilization 0.35 \
  --enforce-eager --swap-space 8 --dtype float16 \
  --mm-processor-kwargs '{"max_pixels": 50176}' \
  --media-io-kwargs '{"video": {"num_frames": -1}}' \
  --reasoning-parser qwen3 --port 8000
```

**Terminal 2 — FastAPI Orchestrator**
```bash
uvicorn orchestrator.main:app --port 7000 --host 0.0.0.0
```

**Terminal 3 — React Frontend**
```bash
cd ui && npm run dev
```

**Terminal 4 — Isaac Sim (GUI mode)**
```bash
conda activate robotic_surgery
PYTHONPATH=$(pwd)/workflows/robotic_surgery/scripts \
  ISAAC_VRAM_FRACTION=0.55 LIGHTWEIGHT_REPLICATOR=1 \
  python isaac_ext/cosmos_bridge_extension.py \
    --env Isaac-Lift-Needle-PSM-IK-Abs-v0 --num_envs 1 --vram-optimized
```

### Diagnostics

```bash
# Check system health
curl http://localhost:7000/health
# → {"isaac_clients": 1, "cosmos_available": true}

# Describe the current scene
curl -X POST http://localhost:7000/describe

# Generate a surgical plan
curl -X POST http://localhost:7000/plan \
  -H "Content-Type: application/json" \
  -d '{"instruction": "Pick up the needle"}'
```

---

## Key Components

| File | Description |
|------|-------------|
| `orchestrator/main.py` | FastAPI hub — WebSocket endpoints, REST API, MJPEG streaming, auto-advance |
| `orchestrator/cosmos_client.py` | Cosmos Reason2 client — plan_action, describe_scene, egocentric/physical reasoning |
| `orchestrator/schemas.py` | Pydantic models — SurgicalAction, SurgicalPlan, IsaacCommand |
| `orchestrator/action_translate.py` | Workspace clamping, skill-based gripper mapping |
| `orchestrator/frame_buffer.py` | WebSocket frame protocol parser |
| `isaac_ext/cosmos_bridge_extension.py` | Isaac Sim bridge — frame streaming, command reception |
| `ui/src/App.jsx` | React UI — live feed, Cosmos chat, Aceternity components |
| `workflows/.../lift_needle_organs_sm.py` | State machine with OrchestratorBridge and CosmosCommandOverride |

---

## Future Implications

### Autonomous Surgical Assistance
As vision-language models scale in capability, SurgiTwin's architecture directly supports progression from advisory AI ("the needle is 2cm to your left") to supervised autonomy ("I'll position the gripper — confirm to grasp"). The command override pattern already supports this transition: the state machine handles routine motions while Cosmos handles complex decision-making.

### Multi-Modal Sensor Fusion
The current system uses RGB frames and kinematic state. The architecture is designed to incorporate additional modalities — depth sensors, force/torque feedback, ultrasound imaging — by extending the WebSocket frame protocol's JSON header. This enables richer physical reasoning about tissue compliance, instrument forces, and subsurface anatomy.

### Sim-to-Real Transfer
Isaac Sim's photorealistic rendering and physics accuracy provide a foundation for sim-to-real transfer. Policies trained and validated in SurgiTwin can be deployed on physical dVRK systems with minimal domain gap, accelerating the path from research prototype to clinical tool.

### Collaborative Multi-Agent Surgery
The orchestrator's WebSocket architecture supports multiple simultaneous robot connections. Future work could coordinate dual-arm PSM configurations or multi-robot teams, with Cosmos reasoning about the joint action space and task allocation across instruments.

### Surgical Training and Education
SurgiTwin can serve as a training platform where surgical residents interact with AI-guided simulation, receiving real-time feedback on instrument positioning, approach angles, and tissue handling — without risk to patients.

### Regulatory Pathway Support
By logging every frame, command, reasoning output, and state transition, SurgiTwin provides the traceability infrastructure required for regulatory submissions (FDA, CE). The deterministic replay capability of Isaac Sim enables reproducible validation of AI-assisted surgical behaviors.

---

## License

This project is part of [Isaac for Healthcare Workflows](https://github.com/isaac-for-healthcare/i4h-workflows) under [Apache 2.0](./LICENSE).

