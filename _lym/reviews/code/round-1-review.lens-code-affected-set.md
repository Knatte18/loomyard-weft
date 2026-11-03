```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, covering what the diff forces outside the lines it touches.

**Per change, the set I found:**

1. **`taskWorktreePresent` now reads an unresolvable path as absent (`wire.go:257`).** I found 15 call sites and judged each.
   - Covered: `wire.go` seams (88, 1113, 1135, 1151), `cancelcmd.go:199` and `:342` (`StopBeam` skips `BeamDown` for a stray directory), `owner.go:240` and `:268`, `startguard.go:319`, `dobbynotice.go:71`, `dobbywire.go:15` and `readopt.go:62` (the `Gone` seam negates it).
   - Covered: `taskWorktreeComplete` doc updated. Tests are in `readopt_integration_test.go` for the stray-path cases, and `wire_test.go` keeps the absent-path assertion.
   - Not covered: the stat-only sibling `presentTaskWorktreePath` / `taskWorktreeLocation` (`wire.go:207-229`). The diff leaves it alone, but its doc says an absent pair is refused by name. For a stray directory, stat succeeds and `lymcwd.ResolveWorktree` returns the bare "not a git repository" error. This reaches `goto.go:19`, which tolerates errors, and `readopt.go:78` (`SpawnPush`), which only warns, and is guarded elsewhere. The new `cli.go:69` sentence says "every batten verb reads it so", which overclaims. See finding A1.
   - Not covered: `beamfork.go:36`. A stray beam directory now gets the "run lym beam create / lym adopt" way forward. The summary itself says this wording is unchecked, and the diff has no test for it. See finding A2.

2. **`Promoted` now keys on the board holding the slug as a task, plus the new `ReachedParent` seam.**
   - Covered: `startguard.go` struct and `refuse` docs, the `cli.go` package doc, and `startguard_test.go` rows for pass and probe error.
   - Covered: `beamland_integration_test.go` pins a pruned slug and a still-a-task slug.
   - Covered: no other `startGuard{` constructor exists outside the tests (`predrive_test.go` and `startguard_test.go:427` leave the new seam nil, which reads as not reached).

3. **`hubChildReachedParent` extracted and shared.** Cancel now uses it, and the guard uses the same probe, so the refusals cannot disagree. Covered by the existing cancel tests.

4. **Refusal pairs that name each other.**
   - The promote refusal (`startguard.go:150`) now fires only when the cancel would proceed.
   - Checked `startguard.go:197` (left-pair refusal, named cancel): with no status found, `ReachedParent` is false unless the parent already holds the work, so no loop.
   - Checked `wire.go:328` (`leftoverWayForward`) and `pairteardown/followup.go:36`: both fire before Land, so no loop.
   - Checked `cancel.go:290`, which names `lym batten run`; the guard now passes that run.
   - Covered.

5. **PATTERN and commits.**
   - The diff touches no `PATTERN.md` or `pattern/` file, and the directive declares no PATTERN change, so there is nothing to land.
   - No commit carries a `Pattern-Departure:` trailer, so nothing to judge.
   - `PATTERN-refusal-way-forward` binds only webster, shed and loom, not `battencli`.
   - No stencils changed.

**Findings**

- **A1 — LOW, consistency.** Location: `internal/battencli/cli.go:69` and `wire.go:207-229`. A stray directory still passes `presentTaskWorktreePath`'s stat and fails in `ResolveWorktree` with the bare "not a git repository". This is the path that `goto` and `SpawnPush` use, so "every batten verb reads it so" is not true. Evidence: `wire.go:217-228`, `goto.go:19`, `readopt.go:78`. Fix direction: have `presentTaskWorktreePath` refuse via `taskWorktreePresent` (so the stray case gets the by-name refusal), or soften the `cli.go` sentence.
- **A2 — LOW, scope.** Location: `internal/battencli/beamfork.go:36-62`. The wording for a stray beam directory is unverified and untested, and the summary lists it under "to check by hand". Evidence: `beamfork.go:50-63`, `summary.md`.

**Tool calls:** none. My context already held every file I needed.
