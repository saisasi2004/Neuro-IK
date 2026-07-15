# NeuroIK — Robotic Arm Studio 🦾

**A desktop app for ML-powered inverse kinematics** — design a robot, generate a dataset, train a neural network, validate it in 3D, and export it for real hardware.

> 📖 For more details, go through the **[documentation](https://saisasi2004.github.io/Neuro-IK/)**.

## Features

- **Robot Selection**: SCARA, Delta, Cartesian, Articulated (2–12 DOF), or **import your own URDF/Xacro**
- **Custom URDF**: Point at a `.urdf`/`.xacro` file or its folder — joints, limits and link lengths populate automatically, and your STL/DAE/OBJ meshes render in 3D
- **Recipe Builder**: Configure links, joint types and limits, with a live 3D preview and per-joint sliders to pose the arm
- **Dataset Generation**: FK-first approach — every sample is reachable by construction
- **ML Studio**: Train 6 architectures (MLP, Transformer, GNN, Diffusion, PINN, RL Agent) with live metrics, loss curves and a target-vs-predicted scatter
- **Simulation Studio**: Interactive 3D viewport with ML inference, analytical-IK comparison (with the formulas one click away) and FK validation
- **Multi-format Export**: ONNX, PyTorch, TF Lite, TensorRT, ROS2, TinyML (C header)
- **Projects**: Recipes, datasets, trained models and exports saved per project — delete any of them from the Dashboard

## Quick Start

**Requirements:** Windows 10/11, Python 3.11+, 8 GB RAM (16 GB recommended). An NVIDIA GPU with CUDA 11.8+ speeds up training; otherwise it falls back to CPU.

Double-click **`NeuroIK.bat`**. That's it.

The first run installs the dependencies and then opens the app. Every run after that goes straight to the app.

> **Note:** the first run shows a console window while it downloads PyTorch and TensorFlow — a few GB, several minutes. Leave it until the app window appears. Later launches are silent.

### Desktop icon (optional)

```bash
python desktop.py --install-shortcut
```

Puts a **NeuroIK** icon on your Desktop that launches the app with no console window.

## Workflow

1. **Projects** → Create a project (or reopen one, with all its state restored)
2. **Select Robot** → SCARA, Delta, Cartesian, Articulated, or **Custom URDF** (enter the path to your `.urdf`/`.xacro` file or its folder)
3. **Prepare Recipe** → Configure the arm, generate a dataset, view workspace analytics
4. **ML Studio** → Pick a model, tune hyperparameters, train with a live dashboard
5. **Simulation** → Test predictions in 3D, validate with FK, measure error
6. **Export** → ONNX, PyTorch, TF Lite, TensorRT, ROS2, or TinyML

## Custom URDF import

Choose **Custom URDF** on the Robot Selection screen and enter a path — either the `.urdf`/`.xacro`
file itself, or **the folder containing it** (best, since mesh paths resolve relative to that folder).

NeuroIK reads the real kinematic chain — every joint's `origin` (xyz + rpy) and `axis` — so an
imported robot runs on a dedicated exact-kinematics engine rather than approximated DH parameters.
Joint types, joint limits and link lengths populate the Recipe Builder automatically, and your CAD
renders in both the Live Preview and Simulation.

- **Mesh formats:** `STL`, `DAE`, `OBJ`. URDF `<box>/<cylinder>/<sphere>` primitives render too.
  Other formats (STEP, IGES) are B-rep CAD that a browser can't tessellate — those links show a
  placeholder and the **kinematics are unaffected**.
- **Xacro:** properties, `${...}` expressions, `<xacro:include>` and `<xacro:if>/<xacro:unless>` are
  expanded internally (no ROS install needed). Files using `<xacro:macro>` are **not** supported —
  pre-expand them with `xacro robot.xacro -o robot.urdf` and import the result.
- **Mimic joints** are dependent, not DOF: they're excluded from the joint count and follow their
  source joint in 3D (e.g. a gripper's second finger).
- **Editing after import:** joint limits and link lengths stay editable — changing a link length
  rescales the spacing to the next joint. CAD meshes are rigid and keep their own size, so a large
  change will show a visible gap. The DOF count and joint types are fixed by the file.

## Documentation

📖 **[Read the full documentation →](https://saisasi2004.github.io/Neuro-IK/)**

Installation, every robot type and architecture explained, the Simulation Studio, and copy-paste integration code for each export format.

## Where files are stored

Everything the app produces lives under a single `data/` folder:

```
NeuroIK/
└── data/
    ├── datasets/   # generated CSVs
    ├── models/     # training checkpoints (best_model.pt)
    ├── exports/    # exported models, one subfolder per format
    └── projects/   # project state (JSON)
```

You can also mirror datasets and exports to any folder you choose — set a **Project Root Folder** on the Dashboard, and files are copied to `<root>/<project name>/{datasets,exports}/`.

## Architecture

```
Robot Config & Recipe  (links, joints, limits)
    ↓
Dataset Generator      (forward kinematics → 100% reachable samples)
    ↓
ML Trainer             (PyTorch, task-space FK loss, live metrics)
    ↓
Simulation Studio      (Three.js 3D + FK validation)
    ↓
Exporter               (ONNX / PyTorch / TF Lite / TensorRT / ROS2 / TinyML)
```

**Why FK-first:** IK is one-to-many — many joint configurations reach the same pose. Sampling joints and computing the pose guarantees valid data, and training against a forward-kinematics (task-space) loss rewards *any* joint vector that reaches the target instead of forcing the network to average conflicting valid answers.

## Tech Stack

- **Frontend**: HTML5, CSS3, Vanilla JS, Three.js, Chart.js
- **Backend**: Python 3.11, FastAPI, PyTorch, NumPy, Pandas, scikit-learn
- **Desktop shell**: pywebview (native window over a local FastAPI server)
- **Design**: Neumorphic light theme, orange accent (`#f27a3a`), Inter font

## Running from source

```bash
pip install -r backend/requirements.txt
python desktop.py                 # native desktop window
```

To run it as a plain web app in your browser instead:

```bash
python -m uvicorn backend.main:app --port 8000
# then open http://localhost:8000
```

> **Note:** NeuroIK is built as a single-user local tool — it keeps active state in memory and can write to folders you choose. Don't expose it to an untrusted network as-is.

## Notes & Gotchas

- **Changed the recipe?** Regenerate the dataset and retrain. Link lengths and joint count define the model's shape, so an old model no longer describes the new robot.
- **Exported model returning nonsense?** ONNX/TF Lite/TensorRT graphs expect a *normalized* pose and return *normalized* joints. Apply the constants from the `model_meta.json` shipped alongside them (see the docs).
- **TinyML** generates a complete, runnable `ik_forward()` for **MLP** models only — other architectures can't reduce to straight-line C. Train an MLP for microcontrollers.
- **TensorFlow** is only needed for the TF Lite export; every other format works without it.
- **Imported URDF shows fewer meshes than the folder holds?** That's expected — NeuroIK draws the
  links the URDF *declares*. Spare STLs shipped alongside it aren't referenced by the description.
