# Stemsplat performance rules

Stemsplat has separate quality lanes:

- reference_safe: default app path. Must preserve existing output within strict drift gates.
- balanced: minor speed improvements with low measured drift.
- turbo: lower quality allowed only when speedup is large and drift is measured/labeled.
- experimental/compiler_lab: isolated exploration only; must not change the default runtime path.

On `main`, only `reference_safe` is implemented. `quality.py` (subcommands
`prepare`, `baseline`, `compare`, `benchmark`, `report`) stamps every report
with `lane: reference_safe` and enforces the gates in `REFERENCE_SAFE_GATES`
against an untouched v0.4.2 baseline; see `quality/README.md`. The `balanced`,
`turbo`, and `experimental/compiler_lab` lanes, their env flags, and the
benchmark harness that measured them exist only on the local archive branch
`archive/2026-11/desktop-iree-wip`. Porting any of them to `main` requires
adding the flag, its own measured drift gates, and promotion tests first.

Performance work must be evidence-based.

Do not claim a speedup unless a benchmark command was run and the result is recorded.
Do not claim quality safety unless the quality harness reports drift metrics.
Do not change default overlap, chunking, precision, backend, or model behavior unless the change is gated behind a flag or explicitly promoted by quality.py gates.

Prefer:
- small measurable changes
- flags for risky behavior
- benchmark reports
- tests for promotion logic
- preserving user-facing defaults

Avoid:
- broad rewrites
- silent fallbacks
- unmeasured quality changes
- toolchain contamination
- making experimental dependencies required for normal installs

Before finishing a performance task, report:
- files changed
- benchmark command/results
- quality command/results
- tests run
- whether the change is reference_safe, turbo, or experimental
- any blocker or next candidate
