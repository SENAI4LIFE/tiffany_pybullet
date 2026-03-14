# Tiffany — Hexapod Simulation (PyBullet)
Simulation of **Tiffany**, a 3-DOF-per-leg hexapod robot.
Hardware repo: https://github.com/Penguin-Lab/tiffany

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

## Prerequisites
- Python 3.x
- pip

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

---

## Controls

| Key | Action |
|-----|--------|
| `E` | Power on — boots the robot and enters idle |
| `R` | Power off — runs shutdown sequence |
| `↑ ↓ ← →` | Walk in the corresponding direction |
| `C` | Switch to **Turn** navigation mode |
| `X` | Switch to **Omni** navigation mode (default) |
| `Z` + `↑ ↓ ← →` | Pose mode — tilt body (pitch / roll) |
| `B` | Rebolar — body sway animation |
| `N` | Balance mode — auto-corrects body tilt |
| `P` or `G` | Give a paw — triggers the *patinha* animation |
| `F` | Toggle camera tracking (follows the robot) |
