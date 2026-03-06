## SurgiTwin: Digital Surgery Twin Powered by NVIDIA Isaac Sim and Cosmos

SurgiTwin is a comprehensive digital twin platform for healthcare robotics, built on NVIDIA's Isaac for Healthcare (I4H) Workflows. It provides end-to-end pipelines for simulating, training, and deploying autonomous robotic systems across surgical and medical imaging applications — from physics-accurate simulation to real-world deployment.

### Key Features

- **Physics-Accurate Simulation** — High-fidelity surgical environments powered by NVIDIA Isaac Sim with RTX ray tracing
- **GPU-Accelerated Ultrasound Simulation** — Custom raytracing-based B-mode ultrasound simulator with tissue-specific acoustic properties
- **Foundation Model Integration** — GR00T N1/N1.5 and PI0 policies for robotic control via imitation learning
- **Sim-to-Real Transfer** — DDS middleware and Holoscan SDK bridge simulation to physical hardware seamlessly
- **Domain Randomization** — NVIDIA Cosmos Transfer 1 for photorealistic rendering and robust policy training
- **Distributed Architecture** — RTI Connext DDS enables multi-machine, real-time communication

### Workflows

| Workflow | Description | Robots |
|----------|-------------|--------|
| **Robotic Surgery** | Surgical task simulation (reach, lift, handover) with RL training | dVRK (PSM/ECM), STAR |
| **Robotic Ultrasound** | Autonomous ultrasound imaging with GPU-accelerated ray tracing sensor sim | Franka + ultrasound probe |
| **Telesurgery** | Low-latency remote surgical operations with haptic feedback | MIRA (Virtual Incision) |
| **SO-ARM Starter** | Surgical assistant for instrument handling with GR00T N1.5 diffusion policy | SO-ARM101 6-DOF |

### Project Structure

```
SurgiTwin/
└── i4h-workflows/
    ├── workflows/
    │   ├── robotic_surgery/        # Surgical simulation & RL training
    │   ├── robotic_ultrasound/     # Autonomous ultrasound pipeline
    │   ├── telesurgery/            # Remote surgery with video streaming
    │   └── so_arm_starter/         # Surgical assistant robot
    ├── tutorials/
    │   ├── assets/                 # Bring-your-own patient/robot/OR guides
    │   │   ├── CT_to_USD/          # CT/MRI to USD conversion
    │   │   ├── bring_your_own_patient/
    │   │   ├── bring_your_own_robot/
    │   │   ├── bring_your_own_or/
    │   │   └── cosmos_transfer1/   # Domain randomization
    │   └── sim2real/               # Sim-to-real deployment guide
    ├── holoscan_i4h/               # Custom Holoscan operators (Clarius, RealSense)
    ├── tools/                      # Environment setup & build scripts
    ├── third_party/IsaacLab/       # Isaac Lab (RL framework)
    └── docs/                       # Documentation
```

### Technology Stack

| Layer | Technologies |
|-------|-------------|
| Simulation | NVIDIA Isaac Sim 5.0, Isaac Lab 2.1–2.3, Omniverse (USD) |
| AI/ML | GR00T N1/N1.5, PI0 (OpenPI), LeRobot, Cosmos Transfer 1, TensorRT |
| Communication | RTI Connext DDS, WebSockets, NVIDIA Holoscan SDK |
| Sensors | GPU raytracing ultrasound, multi-camera RGB/depth, RealSense |
| Deployment | Docker (x86/ARM64), Jetson Orin/Thor, DGX Spark, NVIDIA IGX |

### Workflow Architecture

Each workflow follows a consistent pipeline:

```
Simulation (Isaac Sim)  -->  Training (RL / Imitation Learning)  -->  Inference (TensorRT)  -->  Deployment (Holoscan + DDS)
        |                            |                                      |                           |
   Environments               GR00T / PI0                          Policy Runner              Physical Hardware
   State Machines              RSL-RL / SB3                        Real-time Control           Franka / SO-ARM / MIRA
   Data Collection             Domain Randomization                Latency Benchmarks          Sensor Integration
```

### Capabilities

- **Reinforcement Learning** — RSL-RL and Stable Baselines3 for surgical task training (reach, lift, handover)
- **Imitation Learning** — Demonstration collection and policy training with PI0 and GR00T
- **Teleoperation** — Keyboard, gamepad, SpaceMouse, and OpenXR hand tracking interfaces
- **Synthetic Data Generation** — Automated dataset creation with domain randomization
- **Real-Time Deployment** — Hardware-accelerated video encoding (H.264/HEVC) and sub-25ms inference latency

### Requirements

- **GPU**: NVIDIA with RT Cores (Ampere+), compute capability >= 8.6, 24–30 GB VRAM
- **RAM**: 64 GB+
- **OS**: Ubuntu 22.04/24.04 LTS (x86_64)
- **Software**: Python 3.11, NVIDIA Driver >= 535, CUDA 12.6–12.8, Isaac Sim 5.0

### Getting Started

1. Clone the repository and initialize submodules:
   ```bash
   git clone --recurse-submodules <repo-url>
   ```

2. Set up the environment for your target workflow:
   ```bash
   # Robotic Ultrasound
   source tools/env_setup_robot_us.sh

   # Robotic Surgery
   source tools/env_setup_robot_surgery.sh

   # SO-ARM Starter
   source tools/env_setup_so_arm_starter.sh
   ```

3. Follow the workflow-specific README in `i4h-workflows/workflows/<workflow>/`.

### License

See [LICENSE](i4h-workflows/LICENSE) for details.
