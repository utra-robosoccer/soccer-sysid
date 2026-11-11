# experiments/

Scripts that command the motor on the bench and record data. One script per
experiment protocol; all write CSVs into `data/raw/<name>/` and get an entry in
`data/manifest.yaml`.

| Script | Purpose |
|---|---|
| `record.py` | Run any excitation trajectory (`soccer_sysid/trajectories/`) and record cmd vs. actual to CSV |
| `plot_trajectories.py` | Quick-look plot of recordings (cmd/actual/error) — sanity check before fitting |

Planned protocols (each targets one effect in the model):

- **Chirp / multi-sine** (`sin_time_square`, `sin_sin`) — broadband dynamics, inertia + damping
- **Lift-and-drop / zero-torque** (`lift_and_drop`, `nothing`) — passive dynamics, backdrive friction
- **Constant-velocity sweep** — steady-state friction curve (Coulomb + viscous + Stribeck)
- **Stiction ramp** — slow torque ramp from rest, breakaway torque
- **Step response / latency** — timestamped command step, measure command-to-motion delay

Latency protocols should be run twice: over the direct USB-CAN debugger path
(this repo) and through the full `soccer-firmware` chain (host→USB→master→SPI→slave→CAN,
using its existing `latency.py` tooling) — the difference attributes the firmware
contribution.
