# NeuroIK — Robotic Arm Studio 🦾

**A desktop app for designing, analysing and deploying robot arms** — from a folder of CAD meshes
to a trained model running on an ESP32, and every stage in between.

> 📖 For more details, go through the **[documentation](https://saisasi2004.github.io/Neuro-IK/)**.

NeuroIK has **six workflows**, chosen on the landing page. Each one stands on its own:

| Workflow | Answers |
|---|---|
| **CAD → URDF/Xacro** | *I have meshes and no robot.* Build links and joints, set axes, origins and limits, and generate a validated URDF. |
| **IK Calculator** | *Which joint angles reach this pose?* Exact IK with the algebra shown in your own numbers — closed-form where the geometry allows, numerical where it doesn't, and both compared side by side. |
| **Jacobian & Singularity** | *How well does it move once it's there?* Manipulability, conditioning, the velocity ellipsoid, and where in its travel it loses a direction of motion. |
| **Torque & Dynamics** | *Can the motors cope?* Gravity, inertia, Coriolis and friction per joint, plus payload limits and drive sizing. |
| **Motion & Path Planning** | *What path should it take?* Teach waypoints, shape the path, validate it, export an executable motion program. |
| **ML for IK** | *How do I solve IK fast on hardware that can't run a solver?* Train a network, validate it in 3D, export to 10 formats. |

They share one robot description, so a robot built or imported in one is immediately available to the
rest — with its joint limits, link lengths and mass properties carried through rather than re-entered.
The chosen workflow is stored per project, so reopening one lands in the right studio.

## Features

- **Robot Selection**: SCARA, Delta, Cartesian, Articulated (2–12 DOF), or **import your own URDF/Xacro**
- **Custom URDF**: Point at a `.urdf`/`.xacro` file or its folder — joints, limits and link lengths populate automatically, and your STL/DAE/OBJ meshes render in 3D
- **Recipe Builder**: Configure links, joint types and limits, with a live 3D preview and per-joint sliders to pose the arm
- **Dataset Generation**: FK-first approach — every sample is reachable by construction
- **ML Studio**: Train 6 architectures (MLP, Transformer, GNN, Diffusion, PINN, RL Agent) with live metrics, loss curves and a target-vs-predicted scatter
- **Simulation Studio**: Interactive 3D viewport with ML inference, analytical-IK comparison (with the formulas one click away) and FK validation
- **Multi-format Export**: ONNX, PyTorch, TorchScript, TF Lite, TensorRT, ROS2, TinyML (C header), Keras `.h5`, native `.dll`, Core ML — every one takes a **raw pose** and returns **real joint units**
- **Motion & Path Planning**: Teach waypoints by jogging the arm or dragging a 3D gizmo; pick **Line / Curve / Arc / Spiral** per segment; validate reachability, singularities and obstacle collisions across every interpolated point; preview with transport controls; export **CSV, JSON, Epson SPEL+, G-code or a ROS2 JointTrajectory** node
- **IK Calculator**: Closed-form solvers (SCARA with lefty/righty, planar 2R/3R, gated Pieper for a spherical wrist) and three numerical ones (DLS, pseudoinverse, Newton-Raphson) with multi-start — plus a **live derivation** showing each step with your real substituted numbers, not a generic formula
- **Jacobian & Singularity Studio**: SVD metrics (σ, rank, condition number, Yoshikawa manipulability, isotropy), a velocity ellipsoid in 3D, differential kinematics with damped least squares, and a **singularity map** — a curve for one joint, a log-scaled heatmap for two
- **Torque & Dynamics Studio**: Full rigid-body dynamics via recursive Newton-Euler — `τ = M(q)q̈ + C(q,q̇)q̇ + G(q) + F(q̇)` — with a live torque breakdown, trajectory peak/RMS/energy analysis, payload envelope sweep and motor sizing advice
- **CAD → URDF/Xacro Studio**: Import STL/OBJ/DAE directly (STEP/IGES through FreeCAD), measure volume and bounds, place parts with origin and rotation controls, build the joint tree, enter mass properties, and generate a validated URDF or Xacro package
- **4-view CAD viewport**: one perspective view plus synchronized TOP/FRONT/SIDE orthographic thumbnails, rendered from a single WebGL context
- **Portable build**: package the whole application into a folder that runs on any Windows machine with **no Python installed** — see [Portable build](#portable-build)
- **Projects**: Recipes, datasets, trained models, exports, robot descriptions and mass properties saved per project — delete any of them from the Dashboard

## Quick Start

**Requirements:** Windows 10/11, Python 3.11+, 8 GB RAM (16 GB recommended). An NVIDIA GPU with CUDA 11.8+ speeds up training; otherwise it falls back to CPU automatically.

> **GPU users:** PyPI's default `torch` wheel for Windows is **CPU-only**, so `pip install torch` leaves training on the CPU even with an NVIDIA card installed. `NeuroIK.bat` detects the GPU and installs the CUDA build for you. If you installed by hand, run:
> ```
> pip install --force-reinstall torch --index-url https://download.pytorch.org/whl/cu121
> ```
> ML Studio shows which device it will train on, and tells you if a GPU is present but unusable.

Double-click **`NeuroIK.bat`**. That's it.

The first run installs the dependencies and then opens the app. Every run after that goes straight to the app.

> **Note:** the first run shows a console window while it downloads PyTorch and TensorFlow — a few GB, several minutes. Leave it until the app window appears. Later launches are silent.

### Portable build

Package the app into a folder that runs on any Windows machine with **no Python installed** — copy
it to a USB stick and go.

```bash
python -m venv buildenv
buildenv\Scripts\python -m pip install -r backend/requirements-portable.txt
buildenv\Scripts\python -m pip install torch --index-url https://download.pytorch.org/whl/cpu
buildenv\Scripts\python -m pip install pyinstaller

python build_exe.py --venv buildenv\Scripts\python.exe --zip
```

The result is `dist/NeuroIK/` (**~630 MB**) plus `dist/NeuroIK-portable.zip` (~236 MB). Run
`NeuroIK.exe` inside the folder. Projects, datasets and models are written to a `data/` folder
created **beside the executable**, so the whole folder is self-contained and moves with your work.

> **One real trade.** The portable build uses the **CPU-only** PyTorch wheel — the CUDA one is
> ~4.5 GB against ~200 MB, and needs a matching NVIDIA driver on the target machine anyway. **GPU
> training is therefore unavailable in the portable copy**; use the source install for that.
> TensorFlow and coremltools are also left out, costing the TF Lite, Keras H5 and Core ML export
> formats (3 of 10). Everything else — all six workflows, CPU training of every architecture, ONNX
> and the remaining seven exports — works.

Building from your normal environment instead (`python build_exe.py`) works too, but produces roughly
a **6 GB** folder because of the CUDA wheel. Full details in
[`docs/PORTABLE-BUILD.md`](docs/PORTABLE-BUILD.md).

### Desktop icon (optional)

```bash
python desktop.py --install-shortcut
```

Puts a **NeuroIK** icon on your Desktop that launches the app with no console window.

## Workflow

The **ML for IK** workflow, in full. The other five are shorter — they need a robot and nothing else,
so they run from step 2 with dataset generation and training skipped entirely.

1. **Projects** → Create a project (or reopen one, with all its state restored)
2. **Select Robot** → SCARA, Delta, Cartesian, Articulated, or **Custom URDF** (enter the path to your `.urdf`/`.xacro` file or its folder)
3. **Prepare Recipe** → Configure the arm, generate a dataset, view workspace analytics
4. **ML Studio** → Pick a model, tune hyperparameters, train with a live dashboard
5. **Simulation** → Test predictions in 3D, validate with FK, measure error
6. **Export** → ONNX, PyTorch, TF Lite, TensorRT, ROS2, TinyML, Keras H5, or native DLL

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

Six tabs, one per workflow — each with its own contents list. Installation, every robot type and
architecture explained, the maths behind each analysis studio, what each one deliberately refuses to
do, and copy-paste integration code for every export format.

Building the portable executable is covered separately in
[`docs/PORTABLE-BUILD.md`](docs/PORTABLE-BUILD.md).

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

In a [portable build](#portable-build) that same `data/` folder is created **beside `NeuroIK.exe`**
rather than in the source tree — which is why the whole folder is the portable unit: copy it and your
projects come with it.

## Architecture

```
CAD → URDF Studio      (meshes → links, joints, inertia → validated URDF)
    ↓
Robot Config & Recipe  (links, joints, limits, mass properties)
    ↓
    ├─→ IK Calculator      (exact IK + derivation)
    ├─→ Jacobian Studio    (SVD → conditioning, singularities, velocity)
    ├─→ Dynamics Studio    (RNEA → torques, payload, motor sizing)
    ├─→ Path Planning      (waypoints → validated, timed motion program)
    │
    └─→ Dataset Generator  (forward kinematics → 100% reachable samples)
            ↓
        ML Trainer         (PyTorch, task-space FK loss, live metrics)
            ↓
        Simulation Studio  (Three.js 3D + FK validation)
            ↓
        Exporter           (ONNX / PyTorch / TF Lite / TensorRT / ROS2 / TinyML)
```

One robot description feeds all six. Mass properties entered in the CAD studio are what the Dynamics
studio reads; joint limits set anywhere are the interval the singularity map scans.

**Why FK-first:** IK is one-to-many — many joint configurations reach the same pose. Sampling joints and computing the pose guarantees valid data, and training against a forward-kinematics (task-space) loss rewards *any* joint vector that reaches the target instead of forcing the network to average conflicting valid answers.

## Tech Stack

- **Frontend**: HTML5, CSS3, Vanilla JS, Three.js, Chart.js
- **Backend**: Python 3.11, FastAPI, PyTorch, NumPy, Pandas, scikit-learn
- **Packaging**: PyInstaller (one-folder portable build)
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

## Motion & Path Planning

Pick the workflow on the landing page, choose a robot, and the studio opens at
step 2 — dataset generation and training are skipped entirely, because path
planning needs only link lengths, joint limits and the robot's existing
analytical/numerical IK.

- **Teaching**: jog the joint sliders and press **Add Waypoint** to capture the
  current tool pose, or select a waypoint and drag its 3D gizmo. Rows are
  drag-reorderable and every field is editable inline.
- **Validation** runs IK across *every interpolated point*, not just the
  waypoints — the usual failure is a path whose endpoints are both reachable
  but whose middle is not. It also flags near-singular configurations (by
  Jacobian condition number) and collisions against box/cylinder obstacles,
  checking the arm's **links** as well as the tool.
- **Export is blocked while the path has errors.** A program that drives an arm
  into an obstacle is worth refusing outright.
- **Time parameterization**: trapezoidal (fastest) or S-curve (jerk-limited,
  gentler on gearboxes). Blend radius 0 stops at every waypoint; larger values
  round the corner so the tool carries speed through it.
- **Path IK is seeded** from the previous point's solution. That is ~19x faster
  than solving each point cold, and it keeps consecutive points on the same IK
  branch so the arm cannot flip elbow-up to elbow-down mid-path.

## IK Calculator

One target pose in, every joint solution out — with the algebra that produced it, in **your** numbers.

- **Methods are chosen by the robot, not offered blindly.** A closed form that doesn't exist for your
  geometry isn't listed. SCARA gets an exact solver with lefty/righty branches; a 6-DOF arm with a
  genuinely intersecting spherical wrist gets a Pieper decomposition returning up to 8 solutions;
  everything else falls back to numerical.
- **Every closed-form candidate is verified through FK** before it's offered. During development that
  check caught a wrong Euler extraction by reporting *too few* solutions — the wrong ones never got
  past FK.
- **Convergence is scored on both errors** — `max(pos_err/pos_tol, ori_err/ori_tol)`. Scoring on
  position alone let a solver hit tolerance at iteration 10 while orientation was still 51° out, then
  freeze that iterate and report success.
- **Uncontrollable axes are masked, not down-weighted.** A SCARA can't roll or pitch; that's a
  constraint that doesn't exist, not a small error to weight down.
- **Comparison mode** runs every applicable method on one target — success, solution count, position
  and orientation error, iterations and wall-clock time.

## Jacobian & Singularity Studio

How freely the arm moves from a given configuration, and how close it is to losing a direction of
motion entirely.

- **Finite differences by default**, not autograd. Measured at ~0.69 ms against ~11.6 ms, agreeing to
  ~2 × 10⁻⁸ — and a 25×25 sweep is 625 Jacobians, so that's 0.4 s against 7 s for a difference
  invisible at chart resolution.
- **Structural rows**, found by sampling many configurations rather than inspecting one. A single pose
  can have an accidentally zero row; using it would report a permanent structural fact where there was
  only a momentary one.
- **Never thresholds on `det(J)`** — undefined for any non-square Jacobian, and scale-dependent: a
  determinant of 10⁻⁶ means "singular" in metres and "perfectly healthy" in millimetres. The SVD is
  defined for every shape and the condition number is dimensionless.
- **The singularity map is log-scaled and fitted to the healthy cells.** A linear 1/κ scale is
  unreadable in practice — a healthy arm sits at κ ≈ 5–50, so on a real SCARA scan the brightest cell
  reached only 0.117 of a 0–1 ramp and 76% of cells fell below 0.1.
- **DOF, link lengths and joint travel are editable in place**, because conditioning is a property of
  the mechanism and those are the questions worth asking here.

## Torque & Dynamics Studio

What each joint actually has to deliver — not the static holding torque, which is routinely a fraction
of it.

- **It opens with a gate, and will not invent data.** A NeuroIK recipe carries link lengths and joint
  limits — nothing physical. Mass, centre of mass and inertia are parsed, entered, or estimated from a
  primitive and a density. An estimate keeps its **ESTIMATED** badge everywhere it appears, forever.
- **Total robot mass** can be set from a scale reading: every link rescales by the same factor, which
  is exactly a uniform density change — inertia is linear in mass at fixed geometry, so the tensors
  scale with it and the centres of mass don't move.
- **Friction defaults to zero**, and while it's unset every torque on the page is labelled an
  **ideal-drive lower bound**. A real gearbox adds to it, often a substantial fraction of the no-load
  torque. Size a motor from those figures and it may be undersized.
- **A delta gets no dynamics at all.** RNEA walks a chain base-to-tip and no ordering of a delta's
  links reproduces its kinematics — its three arms share the platform as a closed loop. It declines
  with the reason rather than producing numbers from the wrong model.
- **Verified against known-correct answers**, not merely run: gravity torque vs the closed-form 2R
  solution to 3.5 × 10⁻¹⁵, `τ = Mq̈ + Cq̇ + G` to 8 × 10⁻¹⁶, payload torque vs `−Jᵥᵀ(mg)` to
  4 × 10⁻⁷, energy vs a numerical potential gradient to 7 × 10⁻¹⁰.

## CAD → URDF/Xacro Studio

Where a robot comes from when all you have is meshes. It is a **generator, not a converter** — an STL
knows where the metal is, not where the joints are.

- **Files are read from disk, not uploaded**, so a 200 MB assembly never travels through the browser.
  Browse opens a real OS file dialog in the desktop app, or a built-in folder browser in a browser tab.
  Multi-select imports a whole folder at once.
- **STL, OBJ and DAE read directly; STEP and IGES need FreeCAD** — they're boundary representations,
  not meshes, and tessellating one needs a real CAD kernel. Missing FreeCAD is reported *before* the
  file is accepted, not after.
- **Units are never guessed.** A part exported in millimetres reads as 54 m across; a one-click *treat
  as millimetres* is offered but never applied silently, because guessing wrong shrinks a real gantry
  by a thousand.
- **Placement rotations compose as quaternions**, not by adding 90 to an Euler component — Z 90°
  then X 90° gives rpy (0°, −90°, 90°), not (90°, 0°, 90°). *Centre* and *Sit on z=0* use the bounds
  as currently rotated.
- **Build links from meshes** gives every part a link and chains them — with every joint **fixed and at
  the origin**, deliberately. Inventing plausible revolute axes would fabricate the one thing this
  studio exists to have you specify.
- **Xacro macros are off by default**: NeuroIK's own importer doesn't expand `<xacro:macro>`, so a file
  using them can't be read back by the application that wrote it until you pre-expand it.

## Notes & Gotchas

### Across the workflows

- **The robot is shared, and editing it in one studio changes it everywhere.** DOF, link lengths and
  joint limits are editable from inside the Jacobian and Dynamics studios as well as Robot Selection.
  That is the point — it turns "what if the forearm were shorter" into one field — but the change is
  saved to the project, not scoped to the page you typed it on.
- **Changing DOF or a link length invalidates mass properties.** A different joint count means a
  different number of links, and an estimate was derived *from* the old geometry. Estimates are
  regenerated automatically; values you typed are never silently reshaped — the Dynamics studio drops
  back to its gate and says the properties describe a different robot.
- **A delta is a parallel mechanism, and some things simply do not apply to it.** It gets no dynamics
  (RNEA needs a serial chain) and no DH table. Those studios decline with the reason rather than
  producing numbers from a model that isn't the robot.
- **Estimated mass properties keep an ESTIMATED badge forever**, including inside the generated URDF
  package's README — a URDF has nowhere to record provenance, and a guessed mass produces a guessed
  torque that nothing downstream could distinguish from a measured one.

### ML export

- **Training device** is chosen automatically: CUDA if `torch.cuda.is_available()`, else CPU. No configuration, and no failure if there's no GPU. Explicitly requesting CUDA on a machine without it falls back to CPU rather than crashing. The banner at the top of ML Studio always states which one is in use.
- **Changed the recipe?** Regenerate the dataset and retrain. Link lengths and joint count define the model's shape, so an old model no longer describes the new robot.
- **Normalization is now baked into every export.** Feed a raw pose in metres/radians and read real joint units straight out — no `model_meta.json` arithmetic. `model_meta.json` still ships the constants for reference and sets `"normalization_baked_in": true`; **do not apply them again** or you will get nonsense. The one exception is the ROS2 package, whose node applies them explicitly (its meta says `false`).
- **TinyML** generates a complete, runnable `ik_forward()` for **MLP** models only — other architectures can't reduce to straight-line C. Train an MLP for microcontrollers.
- **Native `.dll`** compiles that same C into a shared library exposing `neuroik_ik_forward(const float *pose, float *joints)` — normalization baked in, no runtime dependencies. It needs a C compiler (MSVC Build Tools or MinGW-w64); NeuroIK finds MSVC automatically even when it isn't on `PATH`. Without one, the export still writes `ik_model.h`, `ik_dll.c` and `build_dll.bat` so you can build it later. MLP/PINN only.
- **Keras `.h5`** rebuilds the network as a native Keras model, so it loads with `tf.keras.models.load_model()` and needs no PyTorch. MLP/PINN only — diffusion, GNN, transformer and RL keep their logic in `forward()` and can't be rebuilt layer-by-layer, so those formats now refuse rather than emitting a wrong model.
- **ROS2** packages ship the weights: `best_model.pt`, a TorchScript `model_traced.pt` and `model_meta.json` all live inside the package, so `ik_node.py` runs after a plain `colcon build` with only torch installed.
- **TorchScript** is the full-precision option for C++ (`torch::jit::load()`): it carries its own architecture, so it needs no model class and no NeuroIK source, and it works for **all six** architectures.
- **Core ML** needs `pip install coremltools` (optional). On macOS/Linux it writes a modern `.mlpackage`; on Windows coremltools ships no native blob writer, so it falls back to the legacy `.mlmodel` — still fine on iOS 13+/macOS 10.15+. Validating by prediction requires macOS, so elsewhere the file is produced but reported unvalidated. Transformer models can't convert (fused attention op); diffusion converts but unrolls all 50 sampling steps into a ~150 MB file.
- **TensorFlow** is only needed for the TF Lite and `.h5` exports; every other format works without it.
- **Export support by architecture:** ONNX, PyTorch, TorchScript, TensorRT, ROS2 and TinyML work for everything. TF Lite, `.h5` and `.dll` are **MLP/PINN only** — the others keep their logic in `forward()` and now *refuse* rather than emitting a wrong model.
- **Imported URDF shows fewer meshes than the folder holds?** That's expected — NeuroIK draws the
  links the URDF *declares*. Spare STLs shipped alongside it aren't referenced by the description.
