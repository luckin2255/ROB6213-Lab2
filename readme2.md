# 🚀 Isaac Lab Go2 Locomotion (Direct RL)

<p align="center">
  <img src="https://img.shields.io/badge/Isaac%20Lab-RL-blue" />
  <img src="https://img.shields.io/badge/Algorithm-PPO-green" />
  <img src="https://img.shields.io/badge/Robot-Unitree%20Go2-orange" />
  <img src="https://img.shields.io/badge/Control-Torque--Level-red" />
  <img src="https://img.shields.io/badge/Framework-RSL--RL-purple" />
</p>

<p align="center">
  <b>Custom Direct Reinforcement Learning Environment for Quadruped Locomotion</b><br>
  Built with NVIDIA Isaac Lab + PPO, featuring torque-level control, reward engineering, and gait-aware learning.
</p>

---

## ✨ Overview

This project implements a **custom locomotion training pipeline** for the **Unitree Go2 quadruped** in NVIDIA Isaac Lab.

Unlike standard manager-based environments, this work uses a **DirectRLEnv design**, giving full control over:

- action → torque mapping  
- reward shaping  
- gait-phase modeling  
- contact-aware learning  
- termination logic  

The project focuses on **how environment design influences emergent locomotion behavior**.

---

## 🧠 Key Contributions

- 🔧 **Direct RL Environment (no manager abstraction)**
- ⚙️ **Explicit PD Torque Control (full actuation control)**
- 📉 **Action Smoothness Modeling (1st & 2nd order penalties)**
- 🦿 **Gait Phase Encoding (clock-based locomotion structure)**
- 🧩 **Contact-Aware Reward Design (force + timing shaping)**
- 📐 **Raibert Heuristic Integration (foot placement learning)**
- 🚨 **Physically Motivated Termination Conditions**

---

## 🏗️ System Architecture
    ┌────────────────────────────┐
    │        Policy (PPO)        │
    │      π(a | observation)    │
    └────────────┬───────────────┘
                 │ actions
                 ▼
    ┌────────────────────────────┐
    │  Action Processing Layer   │
    │  (scale + joint targets)   │
    └────────────┬───────────────┘
                 │ desired q
                 ▼
    ┌────────────────────────────┐
    │  PD Controller (Torque)    │
    │ τ = Kp(q_des - q) - Kd q̇  │
    └────────────┬───────────────┘
                 │ torques
                 ▼
    ┌────────────────────────────┐
    │   Isaac Lab Physics Sim    │
    │   (contacts, dynamics)     │
    └────────────┬───────────────┘
                 │ state
                 ▼
    ┌────────────────────────────┐
    │ Observation + Reward       │
    │ (gait + contact + task)    │
    └────────────────────────────┘

### Why this matters:

- full control over actuator behavior  
- interpretable dynamics  
- enables torque regularization  
- avoids hidden simulator PD  

---

## 👁️ Observation Space

The policy receives:

- base linear velocity (body frame)  
- base angular velocity  
- projected gravity  
- velocity commands  
- joint positions (offset)  
- joint velocities  
- previous actions  
- gait phase signals  

### 🧠 Key Idea

We inject **gait phase (clock inputs)** → allows policy to learn **structured locomotion instead of random stepping**.

---

## 🎯 Command Space

Randomized at reset:

- forward velocity `vx`
- lateral velocity `vy`
- yaw rate `ω`

This forces **multi-directional locomotion generalization**.

---

## 🎮 Reward Design (Core of the Project)

The reward is **fully hand-designed**, combining:

### 🟢 Task Rewards
- velocity tracking (x, y, yaw)

### 🟡 Stability
- orientation penalty  
- vertical velocity penalty  
- angular velocity (roll/pitch)

### 🔵 Smoothness
- action rate penalty  
- action acceleration penalty  

### 🟣 Energy Efficiency
- torque penalty  

### 🔴 Contact-Aware Rewards
- foot clearance  
- contact force shaping  
- gait timing consistency  

---

## 🦿 Gait Modeling

We implement a **clock-based gait generator**:

- cyclic phase variable  
- per-leg sinusoidal encoding  
- stance / swing separation  

This provides:
- temporal structure  
- better learning stability  
- natural coordination  

---

## 📐 Raibert Heuristic (Advanced)

We integrate a **Raibert-style foot placement prior**:

- compute desired foot location  
- compare with actual placement  
- penalize error  

### Effect:
- stabilizes locomotion  
- improves directional control  
- reduces drift  

---

## ⚠️ Termination Conditions

Episodes end early when:

- robot falls (low height)  
- robot flips  
- invalid contact occurs  

This improves:
- training efficiency  
- policy robustness  

---

## 🧪 Training Setup

| Parameter              | Value        |
|----------------------|-------------|
| Algorithm            | PPO (RSL-RL) |
| Simulation Frequency | 200 Hz       |
| Control Decimation   | 4            |
| Environments         | 4096         |
| Episode Length       | 20 sec       |

---

## ▶️ Run Training

```bash
source isaaclab/setup_conda_env.sh

python source/isaaclab_tasks/isaaclab_tasks/scripts/rsl_rl/train.py \
  task=rob6323_go2_flat_direct \
  seed=133581 \
  headless=True


python source/isaaclab_tasks/isaaclab_tasks/scripts/rsl_rl/train.py \
  task=rob6323_go2_flat_direct \
  video=True