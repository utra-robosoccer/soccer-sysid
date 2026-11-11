# fits/

**The output artifact of this repo.** One YAML per motor model with the fitted
actuator parameters, versioned and committed. `soccer-software` vendors these
into `model/source/` (same pattern as its vendored firmware wire headers), and
its generators emit them into the MJCF / ros2_control manifest — so the
ros2_control MuJoCo transport and RL training consume identical actuator models.

Schema sketch (`rs02.yaml`):

```yaml
motor_model: RS02
fit_version: 0.1.0            # bump on any parameter change
provenance:
  dataset_ids: []             # entries from data/manifest.yaml
  fitter_commit: ""           # git hash of this repo at fit time
  date: ""
friction:
  coulomb_nm: null
  viscous_nms: null
  stribeck_nm: null           # optional
  stribeck_vel_rads: null
armature_kgm2: null
torque_constant_scale: null   # measured / reported
latency:
  delay_ms: null
  delay_steps_at_hz: {}       # e.g. {100: 1, 500: 4}
limits:
  tau_nm: 17.0
  vel_rads: 44.0
```

Rules: never edit a fit in place after soccer-software has pinned it — bump
`fit_version` and cut a new file state, mirroring the firmware tag convention.
