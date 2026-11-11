# soccer-sysid

Actuator system identification for the RoboSoccer humanoid. This repo answers
one question end-to-end: **given a command trace, how close is MuJoCo to the
real motor — and how much closer does each modeled effect (friction, firmware
latency, ...) bring it?**

Pipeline: command and record RobStride motors on the bench (direct USB-CAN,
bypassing the robot stack) → fit each effect mathematically → model it in
MuJoCo → replay the same commands in sim vs. real and measure the error.

The output artifact is `fits/<motor_model>.yaml` — fitted actuator parameters
that `soccer-software` vendors into its `model/` SSOT, so the ros2_control
MuJoCo transport and `soccer-rl` training consume identical actuator models.

## Repo map

```
soccer_sysid/            # installable package (pip install -e .)
├── drivers/             # RS02 CAN framing + motor class (CH341 USB-CAN debugger)
├── trajectories/        # excitation signals (chirp, multi-sine, lift-and-drop, ...)
└── cli.py               # direct motor control CLI (`rs02 ...`)
experiments/             # bench protocols: record trajectories to data/raw/   → README
data/
├── raw/                 # recordings (gitignored — backed up out of git)
└── manifest.yaml        # tracked provenance for every dataset
fitting/                 # offline parameter estimation per effect             → README
sim/                     # single-motor / pendulum MuJoCo models w/ effects    → README
validation/              # sim-vs-real replay + error metrics (thesis result)  → README
fits/                    # OUTPUT: versioned per-motor parameter YAMLs         → README
notebooks/               # exploration + thesis figures
figures/                 # rendered figures referenced by the thesis
```

## Place in the stack

| Repo | Role | Interface to this repo |
|---|---|---|
| `soccer-firmware` | robot firmware + host link | full-chain latency measurements (its `latency.py`); direct-CAN vs. full-chain delta = firmware latency contribution |
| `soccer-software` | ROS2 stack + robot model SSOT | consumes `fits/*.yaml` into `model/source/` (vendored, pinned by fit_version) |
| `soccer-rl` | MuJoCo Playground training | consumes soccer-software's generated MJCF — inherits the fits transitively |

Experiments here use the **CH341 USB-CAN debugger directly** (no firmware in the
loop) to isolate motor-level effects from transport effects. Characterize the
debugger's own latency first — it retransmits and is not transparent.

## Setup

**CH341 Linux kernel driver** — install before plugging in the adapter:

```bash
git clone https://github.com/WCHSoftGroup/ch341ser_linux
cd ch341ser_linux
make && sudo make install
```

The adapter then appears as `/dev/ttyCH341USB0`.

**Python environment:**

```bash
python3 -m venv .venv && source .venv/bin/activate
pip install -e ".[sim,fit]"
```

## Quick start

```bash
# 1. Check and zero the motor
rs02 read 1
rs02 init 1
rs02 zero 1 --rate 0.3 --set-mech-zero

# 2. Run an identification trajectory (records to data/raw/<traj>/)
python experiments/record.py sin_time_square    # primary chirp, 30 s
python experiments/record.py lift_and_drop      # free-fall inertia/friction
python experiments/record.py nothing            # zero-torque backdrive baseline

# 3. Quick-look plot
python experiments/plot_trajectories.py --traj sin_time_square
```

After a recording session, add an entry to `data/manifest.yaml` (dataset id,
rig, motor serial, trajectories) — fits only cite manifested datasets.

## Trajectory reference

| Name | Formula | Duration | Targets |
|---|---|---|---|
| `sin_time_square` | A·sin(k·t²) | 30 s | chirp — broadband dynamics |
| `sin_sin` | sin(t)·π/2 + sin(5t)·0.5·sin(2t) | 30 s | multi-frequency, non-periodic |
| `lift_and_drop` | cubic to −π/2, then torque off | 6 s | passive dynamics, free-fall |
| `up_and_down` | cubic: 0→π/2→0.8·π/2 | 6 s | gravity loading both directions |
| `nothing` | 0, torque off | 6 s | backdrive friction baseline |

Recording CSV columns: `t_s, cmd_rad, act_rad, err_rad, vel_rads, torque_nm, temp_c`.

## Direct motor control (rs02 CLI)

```bash
rs02 read 1                          # single feedback read
rs02 read-loop 1 --hz 50             # continuous poll
rs02 goto 1 1.57 0.3                 # ramp to position
rs02 sine 1 --amp 0.2 --freq 0.25    # sine sweep
rs02 zero 1 --rate 0.3               # slow ramp to zero
rs02 set-zero 1                      # set mech zero at current pos
rs02 set-id 1 2                      # change CAN ID
rs02 off 1                           # disable output
```

Port defaults to `/dev/ttyCH341USB0` or the `RS02_PORT` environment variable.

## CAN protocol

Communication is CH341 serial at **921600 baud**. Frames use the RobStride RS02
extended-ID format:

| Direction | CAN ID | Payload |
|---|---|---|
| TX | bits[28:24]=comm_type, [23:8]=torque_ff, [7:0]=motor_id | 8 B: pos[2] vel[2] kp[2] kd[2] |
| RX | bits[28:24]=COMM_FEEDBACK, [15:14]=mode, [13:8]=faults, [7:0]=motor_id | 8 B: pos[2] vel[2] torq[2] temp[2] |

RS02 physical limits: ±12.57 rad, ±44 rad/s, ±17 Nm, Kp 0–500, Kd 0–5.
