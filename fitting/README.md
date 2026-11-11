# fitting/

Offline parameter estimation: takes recordings from `data/raw/`, produces fitted
parameters in `fits/`.

Planned fitters, one per modeled effect:

- **Friction** — Coulomb + viscous (+ Stribeck if the constant-velocity data
  supports it): `tau_f = tau_c*sign(v) + b*v [+ (tau_s - tau_c)*exp(-(v/v_s)^2)*sign(v)]`
- **Rotor inertia / armature** — reflected inertia from chirp + drop tests
- **Torque constant / torque-reporting accuracy** — commanded tau_ff vs. measured
  torque vs. external reference (load cell / pendulum gravity model)
- **Latency** — command-to-motion delay distribution from step responses, fitted
  as an n-step delay at the target control rate

Conventions:

- Every fitter is a script/notebook that reads a dataset listed in
  `data/manifest.yaml` and writes a `fits/<motor_model>.yaml` section plus a
  residual plot into `figures/`.
- Record the dataset id and fitter git hash in the fit file — a fit without
  provenance can't go into the thesis or into soccer-software.
