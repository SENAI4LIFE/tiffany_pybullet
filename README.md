# Tiffany: Hexapod Simulation (PyBullet)

Simulation of **Tiffany**, a 3-DOF-per-leg (coxa/femur/tibia) hexapod robot, built on **PyBullet**.
Hardware repo: https://github.com/Penguin-Lab/tiffany

[![Python](https://img.shields.io/badge/Python-3-yellow?logo=python)](#)
[![PyBullet](https://img.shields.io/badge/PyBullet-Physics-4B8BBE)](#)

---

## Table of Contents

- [Overview](#overview)
- [Requirements](#requirements)
- [Features](#features)
- [Quick Start](#quick-start)
- [Install](#install)
- [Run](#run)
- [Controls](#controls)
- [Terrain](#terrain)
- [Modeling](#modeling)
- [Gait and Kinematics](#gait-and-kinematics)
- [Project Structure](#project-structure)
- [Design Decisions](#design-decisions)

---

## Overview

This repository simulates Tiffany, a 6-legged (hexapod) robot, standalone in PyBullet: a self-contained gait engine drives all 18 leg joints directly from keyboard input, with no external middleware. It computes real-time per-leg forward/inverse kinematics and Bezier-curve leg trajectories, and generates a randomized obstacle course to exercise the gait against varied terrain.

<p align="center"><em>Provenance: this repository is where Tiffany's hardware gait algorithms and kinematics were first converted to Python and developed against the 3D model. It served as the basis for <a href="https://github.com/SENAI4LIFE/tiffany_gazebo">SENAI4LIFE/tiffany_gazebo</a>, a later ROS 2 / Gazebo simulation with a full navigation stack, where the gait engine and inverse kinematics were substantially refined. Those refinements (smoothstep-eased swing trajectories, corrected body-tilt kinematics, and improved turning geometry) have since been ported back into this repository.</em></p>

### What the robot can do

- Walk and turn using body-frame Bezier-curve leg trajectories, in either omnidirectional or car-like turning navigation modes.
- Boot/shutdown through a smooth stand-up/sit-down sequence.
- Balance its body against terrain tilt using the simulated base orientation.
- Pose: manually tilt the body (pitch/roll) in place.
- Perform two idle animations (**Rebolar**: a hip-sway wiggle, **Patinha**: a "paw" gesture).
- Traverse a procedurally generated obstacle course (slopes, stairs, stepping stones, logs, beams, and more).

## Requirements

- Python 3.x
- pip

## Features

| Feature | Description |
|---|---|
| **Gait engine** | Bezier-curve leg trajectories with per-leg forward/inverse kinematics and smoothstep easing (`main.py`) |
| **Two navigation modes** | Omnidirectional walking and car-like turning, toggled with `X`/`C` |
| **Body posing** | Manual pitch/roll pose mode, and an auto-leveling Balance mode driven by the simulated IMU-equivalent base orientation |
| **Idle animations** | Rebolar (hip-sway) and Patinha (paw) gesture, both built on the same body-posing and per-leg-trajectory primitives as walking |
| **Boot/shutdown sequences** | Smooth interpolated stand-up/sit-down between a stowed pose and the walking stance |
| **Procedural terrain** | A configurable-difficulty obstacle course (`bootcamp.py`) covering ten distinct prop types |
| **Keyboard teleop** | Full manual control via PyBullet's built-in keyboard event polling, no external input device required |

---

## Quick Start

```bash
git clone https://github.com/SENAI4LIFE/tiffany_pybullet.git && cd tiffany_pybullet
python3 -m venv venv && source venv/bin/activate
pip install --upgrade pip pybullet numpy
python3 main.py
```

See [Install](#install) for full details and [Controls](#controls) for the full key reference.

---

## Install

**1. Clone the repository**
```bash
git clone https://github.com/SENAI4LIFE/tiffany_pybullet.git
```
**2. Create a virtual environment**
```bash
python3 -m venv venv
```
**3. Activate the virtual environment**
```bash
source venv/bin/activate
```
**4. Install dependencies**
```bash
pip install --upgrade pip pybullet numpy
```
**5. Navigate to the project directory**
```bash
cd ~/tiffany_pybullet/
```

---

## Run

```bash
source venv/bin/activate
python3 main.py
```

This opens the PyBullet GUI with Tiffany standing on a ground plane, followed by a procedurally generated obstacle course (see [Terrain](#terrain)). The robot starts `POWERED_OFF`; boot it with `E` before driving it.

---

## Controls

| Key | Action |
|-----|--------|
| `E` | Power on: boots the robot and enters idle |
| `Q` | Power off: runs shutdown sequence |
| `↑ ↓ ← →` | Walk in the corresponding direction |
| `C` | Switch to **Turn** navigation mode (car-like: forward/backward walks straight, turning arcs each leg's step) |
| `X` | Switch to **Omni** navigation mode (default: strafes directly toward the pressed direction) |
| `Z` + `↑ ↓ ← →` | Pose mode: tilt body (pitch / roll) |
| `R` | Rebolar: body sway animation |
| `B` | Balance mode: auto-corrects body tilt |
| `P` | Give a paw: triggers the *patinha* animation |
| `F` | Toggle camera tracking (follows the robot) |

---

## Terrain

`bootcamp.py` procedurally lays out a lane of ten obstacle props after the ground plane loads: `slope`, `block`, `stepping` (stones), `stairs`, `roundy` (a buried sphere), `pyramid`, `logs`, `slant`, `beams`, and `canyon` (a gap between two platforms). Each prop's height scales with a single `DIFFICULTY` constant, making the whole course easier or harder to traverse without touching individual prop definitions.

---

## Modeling

The robot was designed in Autodesk Fusion, exported as `.step`, then assembled and mate-constrained in Onshape.<br>
[onshape-to-robot](https://github.com/Rhoban/onshape-to-robot) was used to convert the Onshape assembly into URDF.
To regenerate the URDF from your own model, set up `keys` and `config.json` <br>
as described in the [onshape-to-robot docs](https://onshape-to-robot.readthedocs.io/), then run:
```bash
pip install onshape-to-robot
source keys
onshape-to-robot <path-to-config-folder>
```
`keys` file:
```bash
export ONSHAPE_API=https://cad.onshape.com
export ONSHAPE_ACCESS_KEY=<your-access-key>
export ONSHAPE_SECRET_KEY=<your-secret-key>
```
`config.json`:
```json
{
    "documentId": "<your-document-id>",
    "outputFormat": "urdf",
    "assemblyName": "robot",
    "ignore": {
        "corpo_center": "collision"
    }
}
```

---

## Gait and Kinematics

Each leg is a 3-DOF serial chain (coxa → femur → tibia) with link lengths `L1 = 2.56`, `L2 = 9.00`, `L3 = 12.16` (cm, matching the URDF/mesh scale). `fk()`/`ik()` implement forward and inverse kinematics for a single leg from those lengths; every gait function ultimately produces a target foot XYZ per leg, which `ik()` converts to the three joint angles sent to `set_leg()` (`p.setJointMotorControl2` in `POSITION_CONTROL` mode, one call per joint).

Walking and turning trajectories are built from cubic Bezier swing curves (`build_bezier_points`, `trajetoria_linear`) sampled over a 25-point cycle (`TOTAL_PONTOS`) with smoothstep easing, and legs are grouped into two opposing tripods via a fixed per-leg phase offset (`OFFSETS`): a standard alternating tripod gait. `compute_turn_1` blends each leg's straight-line step with a circular (in-place-rotation) step (`mapeia_circular`), using an arc-corrected step for the four corner legs so the whole body rotates about its center rather than skidding. `compute_ik_corpo` handles whole-body roll/pitch posing (used by `Balance` and `Pose` mode): it rotates each leg's shoulder mount and re-solves the foot position needed to keep that foot planted at its original world location. `Rebolar` and `Patinha` are pre-scripted animations built on top of the same body-posing and per-leg-trajectory primitives.

Boot and shutdown are smooth 50-step interpolations between a stowed/folded pose and the standing/idle pose (raising or lowering the whole body in two phases: first horizontal positioning, then vertical), rather than an instantaneous joint snap.

---

## Project Structure

```
tiffany_pybullet/
├── main.py           # Gait engine, kinematics, keyboard control loop, simulation setup
├── bootcamp.py        # Procedural obstacle-course terrain generation
├── robot.urdf          # Robot model (generated via onshape-to-robot, see Modeling)
├── assets/               # STL meshes and Onshape part references used by robot.urdf
└── README.md
```

## Design Decisions

- **Direct joint control in a single Python process, instead of a middleware layer.** With no ROS/Nav2 stack to talk to, the keyboard-to-joint-angle pipeline is a single tight loop: read keys, pick a state, compute six foot targets, convert to joint angles, send position commands. This keeps the simulation trivially easy to run (`python3 main.py`, no build step, no launch files).
- **A single `DIFFICULTY` scalar for the whole obstacle course**, rather than per-prop tuning knobs, so the course can be made harder or easier for gait testing with a one-line change.
- **Degrees, not radians, as the kinematics interface.** `fk()`/`ik()` take and return degrees so the constants in `LEG_CONFIGS`/`ANGLES_STOW_BY_LEG` read directly as joint angles; conversion to radians happens once, at the `p.setJointMotorControl2` call site.
