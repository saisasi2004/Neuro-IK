# NeuroIK — Robotic Arm Studio 🦾

**A desktop app for ML-powered inverse kinematics** — design a robot, generate a dataset, train a neural network, validate it in 3D, and export it for real hardware.

> 📖 For more details, go through the **[documentation](https://saisasi2004.github.io/Neuro-IK/)**.

## Features

- **Robot Selection**: SCARA, Delta, Cartesian, Articulated (2–12 DOF)
- **Recipe Builder**: Configure links, joint types and limits, with a live 3D preview
- **Dataset Generation**: FK-first approach — every sample is reachable by construction
- **ML Studio**: Train 6 architectures (MLP, Transformer, GNN, Diffusion, PINN, RL Agent) with live metrics, loss curves and a target-vs-predicted scatter
- **Simulation Studio**: Interactive 3D viewport with ML inference, analytical-IK comparison and FK validation
- **Multi-format Export**: ONNX, PyTorch, TF Lite, TensorRT, ROS2, TinyML (C header)
- **Projects**: Recipes, datasets, trained models and exports saved per project

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
2. **Select Robot** → SCARA, Delta, Cartesian, or Articulated
3. **Prepare Recipe** → Configure the arm, generate a dataset, view workspace analytics
4. **ML Studio** → Pick a model, tune hyperparameters, train with a live dashboard
5. **Simulation** → Test predictions in 3D, validate with FK, measure error
6. **Export** → ONNX, PyTorch, TF Lite, TensorRT, ROS2, or TinyML

## Documentation

📖 **[Read the full documentation →](https://saisasi2004.github.io/Neuro-IK/)**

Installation, every robot type and architecture explained, the Simulation Studio, and copy-paste integration code for each export format.

Also available offline — open `docs/index.html` in any browser.

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

