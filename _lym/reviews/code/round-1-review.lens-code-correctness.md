```yaml
lens: code-correctness
outcome: reported
```

Lens list (code-correctness): no defects found.

- `pawl-recipe.yaml` `Preflight` row, engine `Preflight`, `on_done: Beam-Up`, `entry: Preflight`. I ran a pawl seed through `wire` (`internal/loomcli/wiring.go:556`). `Env.Cwd` is set unconditionally from `cwd` before the pawl branch at line 833, so `requireAbsRoot("Preflight","Cwd",…)` sees the absolute cwd that `lymcwd.CwdFrom` returned. `configRejectUnknown` gets an empty config. Correct.
- Seed against recipe: the shared seed sets `CurrentProducer` to `Preflight` (`internal/loomshed/seed.go:57`), and `Preflight` is now a recipe row, so the stop in the board entry no longer fires. Correct.
- Preflight failure path: the row has no `on_stuck`, same as darn and loom. A failure halts the run with the failure list and a way forward. Correct and consistent with D1.
- `holdFreeProducers` with `Preflight` added (`internal/boardengine/runlock.go:42`), checked on four inputs:
  - a beam project at `running · Preflight` gives `RunHolds` false;
  - a beam project at `blocked · Preflight` (a halt) gives false, as the doc says;
  - a non-beam project or a task at `Preflight` gives true, because the `Kind == KindProject && Beam` conjunction fails;
  - a loom or darn task at `Preflight` still holds its entry.

  All correct and within D2's bound.
- `runTakesNoSlot` (`internal/battencli/dobbyreads.go:180`) reads `RunHolds`, so a beam run at `Preflight` takes no slot. It matches the doc comment, and it is the same behavior as at `Beam-Up`. Correct.
- Resume paths: a run halted at the old seed `Preflight` now resumes at the new row. A run past its seed keeps a row the recipe still names, since nothing was renamed or removed. Correct.
- Preflight in an unmarked beam pair: the open risk the record names. A grep of `internal/preflight` shows no beam or marker predicate, so the checks do not depend on `Beam-Up`'s mark. A dirty pair stops at `Preflight` with the failing check named, as the record bounds. I did not read the check bodies, so I cannot confirm that the checks pass in a fresh beam pair.
- Test edits (`runlock_test.go`, `wiring_beam_test.go`, `pawlrecipe_test.go`): the new rows and the `Entry` assertion match the recipe's behavior as I traced it. Correct.

No findings.

Tool calls (1 of 6):
1. `grep` over `internal/preflight/*.go` for beam or marker dependence. The manifest did not cover the check bodies.
