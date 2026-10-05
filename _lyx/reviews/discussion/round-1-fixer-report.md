# Fixer report — discussion round 1

All five findings fixed in `_lyx/discussion/decision-record.md`; `support-log.md` is unchanged (its Interview and Rejected alternatives already hold the narrative moved out of D1's Rationale).

## Changed

- F1: D3 now says the three keys are read from `envelope_path`, or from the printed envelope when it carries none (the D2 fallback); the stencil acceptance bullet requires the stencil to say so.
- F2: D3 gained a `Bound:` paragraph for the no-`cd` rule (a bare call resolves the run from the cwd's own seed, so it refuses or steps a run seeded there under the same run-id) and keeps the verbatim-refusal sentence.
  The Open risks entry points at that bound.
  The stencil acceptance bullet now requires the never-change-cwd rule.
- F3: D2 Rationale reads "at a stop or a repair" instead of "at parking or repair".
- F4: the no-record acceptance bullet covers a failed record write as well as an empty `StepsDir`.
- F5: D1 Rationale lost the `stdin: "pipe"` / `bash -c` mechanics and the `BASH_ENV` comparison, keeping why the env file works; the Auto-mode assumption drops "once per session" and points at the first Open risk; the Notes bullet keeps the exact prelude shape and says the integration test reproduces it.
  No requirement was lost: the prelude shape stays in Notes and Acceptance 3, and the child-`bash` behavior stays in D1 and Acceptance 3.

## Deferred

None.
