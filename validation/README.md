# validation/

Sim-vs-real comparison — the thesis result. Same command trace into the real
motor (recorded in `data/raw/`) and into each `sim/` model layer, then error
metrics per layer:

- position / velocity trajectory RMSE and peak error
- torque profile error
- frequency-domain match (chirp runs)
- delay alignment (cross-correlation lag)

Output: a table per motor model of "fidelity improvement per modeled effect"
(baseline → +friction → +latency → ...), plus the figures that go in the thesis.
Keep every metric script deterministic and runnable from a dataset id in
`data/manifest.yaml` so results regenerate from scratch.
