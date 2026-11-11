# sim/

MuJoCo models of the bench rigs with the fitted effects applied — single motor
and single-motor pendulum, not the full robot.

Contents (as they land):

- `pendulum.xml` — bench rig MJCF: one RS02 + arm/mass matching the physical rig
- Actuator model layers, each toggleable so fidelity gains are attributable:
  1. baseline (ideal torque actuator / plain position servo)
  2. +friction (`frictionloss`, `damping`, `armature` or custom force callback)
  3. +latency (n-step action delay queue)
  4. +torque/velocity limits, encoder quantization, etc.
- A replay runner: feeds a recorded command trace from `data/raw/` into the sim
  at the same rate it was executed on hardware.

The full-robot model stays in `soccer-software/model/` (the SSOT); this directory
only proves effects out at single-joint scale before parameters are exported.
