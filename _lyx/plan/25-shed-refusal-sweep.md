# Card 25 — shed-refusal-sweep

**Edit:**
- `internal/shedengine/run.go`
- `internal/shedrun/seed.go`
- `internal/shedcli/seed.go`
- `internal/shedengine/step_test.go`
- `internal/shedrun/seed_test.go`
- `internal/shedcli/seed_test.go`

**Intent:**
Give every remaining shed refusal a way forward (D1) and a reaching test, in the trailing `way forward: …` form.
The `goto` refusals and the two stops card 19 fixed are already done.

Messages to fix:

- `shedengine` `Run` and `Step`, `ErrShedBusy`: the wrap names `lyx shed pause` (asks the live driver to stop at its next producer boundary) and `lyx shed status` (shows the holder).
  A module that supplies its own busy text through `Spec.RunBusyMessage`/`StepBusyMessage` or a `PreStep` hook keeps it.
- `shedengine` step, status file missing while its seed exists: seed or bootstrap the run through its recipe's bootstrap verb, or `lyx shed seed` for a recipe without one.
- `shedrun` `ErrDisagreeingSeed`: keep the existing seed and drive it (`lyx shed status <run-id>` shows it), or address a different run-id.
- `shedcli` seed, "refuses to write here": run `lyx shed seed` from the worktree the recipe drives, which the wrapped cause names.
- `shedcli` seed, recipe without a bootstrap verb under `--driver llm`: re-run with `--driver go`.

Already carrying a way forward, test-only where untested: the missing-seed refusal (`shedrun.MissingSeedMessage`), status's `--watch --json` refusal, and the absent-status refusals each recipe tells through `Spec`.

Grouped: `Shed` validation failures (wiring), an invalid persisted `state` or an undecodable status file (unreachable from a file Shed wrote), an unrecognised producer outcome (a broken adapter), a vanished status file mid-run and lock-directory creation (raw I/O), and usage refusals that already list the legal values (malformed `--param`, unknown driver or recipe, invalid run-id).

Tests: each fixed row is reached from a status or seed file as the verbs write it, the message asserted to contain its way forward, the way forward taken (release the held lock, write the seed, drive the existing seed, re-run with `--driver go`), and the refused call asserted to succeed.
Any test that spawns git carries the `integration` tag.

**ImpactSummary:** Message text and tests only in shedengine, shedrun and shedcli; sentinels and envelope keys are unchanged.
