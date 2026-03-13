# Tiffany — Hexapod Simulation (PyBullet)
Simulation of **Tiffany**, a 3-DOF-per-leg hexapod robot.
Hardware repo: https://github.com/Penguin-Lab/tiffany

---

## Modeling
The robot was designed in Autodesk Fusion, exported as `.step`, then assembled and mate-constrained in Onshape.<br>
[onshape-to-robot](https://github.com/Rhoban/onshape-to-robot) was used to convert the Onshape assembly into URDF.

To regenerate the URDF from your own model, set up `keys` and `config.json` as described in the [onshape-to-robot docs](https://onshape-to-robot.readthedocs.io/), then run:

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

**2. Navigate to the project directory**
```bash
cd ~/tiffany_pybullet/
```

**3. Create a virtual environment**
```bash
python3 -m venv venv
```

**4. Activate the virtual environment**
```bash
source venv/bin/activate
```

**5. Install dependencies**
```bash
pip install --upgrade pip pybullet numpy
```

---

## Run
```bash
source venv/bin/activate
python3 main.py
```
