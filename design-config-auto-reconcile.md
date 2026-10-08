# Retired config keys migrate themselves

`config-auto-reconcile` · Note · enhancement, config

From webster-dead-ends PR-Gate (2026-10-08): that task retired the batcher keys `master_base` and `startup_context`, and Loom-Preflight now refuses a `batcher.yaml` that still carries them, pointing to `lyx config reconcile --apply`.
Every hub and every in-flight pair then needs a manual reconcile after the deploy.

- Loom-Preflight could run the batcher migration itself, or batten could reconcile a seeded pair config, so a retired key never needs a manual reconcile.
- `merriamFixedContext` is a constant from one measurement, so the system-prompt-and-tools figure drifts silently as Claude Code changes; a calibrate step in `tools/tokencount` could re-measure it.
