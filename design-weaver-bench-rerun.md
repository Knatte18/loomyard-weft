# Bench Weaver against Webster again on the same three landed plans, both run side by side the same day

`weaver-bench-rerun` · Task · research, weaver, webster · running · Darn-Write

## Why

The bench of 2026-11-27 (`docs/research/weaver/bench-2026-11-27/report.md`) had only one plan where Weaver and Webster ran the same day,
so its quality and cost comparison is thin.
It is run again after Weaver's four named changes have landed (operator, 2026-11-28).

## Runs

Exactly six implementer runs and six reviews:

- The same three plans as the first bench: `board-viewer-server`, `launch-manifest-parity` and `gate-test-map-mode`, from the same base commits.
- Each plan once with Weaver on its default ladder and once with Webster on its default models, the two started together, the same day.
- One Burler round per result, as in the first bench: a `sonnet[medium]` reviewer with the code lens fan and the archived record and plan as the answer key.

Set up each run the way the first bench did, in a worktree pair of its own.

## Report

A new report beside the first one, with the same tables: wall time, gate time, fixer time, dollars, ladder and stops, verify, diff and review findings.
It checks that the four changes show: no attempt killed by the shuttle timeout, deviations in `summary.md`, and `gate_spans` on the Weaver rows.
It explains where Weaver's saving comes from, measured rather than guessed: cache reads per batch for Weaver against cache reads per fork for Webster.
It ends with a recommendation on making Weaver tapestry's default implementer.
