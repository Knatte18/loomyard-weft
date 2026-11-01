```yaml
lens: guard-reach
outcome: reported
```

guard-reach lens: whether each guard, refusal, branch and test row the cards add can be reached from a real entry point.

**Per guard:**

- Card 1: none (pure string forms).
- Card 2, `buttons: null` worktree-file refusal (FillMissing shape mismatch): reachable. A mapping group in the reed worktree file loads: reachable, since the template's sequence is not descended (`internal/yamlengine/fill.go:83`). Machine `{task: {}}` gives "different shape": reachable. Machine `{task: {label: x}}` gives "not in the desktop template": reachable (`internal/configengine/layers.go:341-347`). Local `{task: []}` refusal: reachable.
- Card 3:
  - `placeButtons` group not a sequence: reachable from reed's hub base or worktree file and desktop's hub file.
  - Same from `desktop.machine.yaml`: unreachable, because the load refuses it first (layers.go:345-347). The render then only ever gets a zero node. This is not a defect, since the guard is still reached from other files.
  - Mistyped entry: reachable.
  - Each `buttonEntryProblem` reason: reachable from YAML.
  - `prime` group not a sequence: reachable from reed.yaml.
  - Zero node: reachable from a hand-built Config or a failed desktop load.
- Card 4:
  - Unreadable origin gives a closure error: reachable.
  - Failed desktop load Warn: reachable.
  - Task `Beam` absent-anchor branch (empty `Anchor`): reachable in production. **The tier1 row meant to reach it does not** (finding 1).
  - Live-other-pair branch in the same test: **not reached**, same cause (finding 1).
  - Post-overwrite `ParentOf` read error: reachable.
- Card 5: `FailedFormatRepinIsNonFatal` reachable through the fake stale stamp. The no-VCS-stamp and failed-read early returns are unchanged.
- Card 6:
  - Nil closure: reachable (standalone).
  - Closure error: reachable.
  - Unresolvable shell: reachable at attach, where `pinGeometryOptionsLocked` runs with `shellPath ""` (`internal/reedengine/attach.go:94-98`). At boot, `preflightBootHost` refuses it first (`internal/reedengine/lifecycle.go:330-333`). Reachable through attach, so held.
  - `Beam` error: reachable.
  - Empty `Anchor`: reachable.
  - Failing invocation: reachable.
- Card 7: empty-`Name` unset: reached by the unit row. A production strand with an empty name was not confirmed, so I report nothing.
- Card 8:
  - Step 1, empty, non-JSON or unreadable spec: reachable by a direct call or a click on a stale-drawn bar. An empty render clears the row and the spec together.
  - Step 2, no entry or non-numeric suffix: the non-numeric suffix is reachable only by a direct call, since card 9's `#{m:lymbtn*,…}` admits it. The missing entry is reachable by direct call or a draw/render race. Held.
  - Step 3, unconfirmed empty `@lym_strand_name`: reachable on a pane an older lym launched. Confirmed `agentname.Parse` failure: reachable only by a direct call; the test row covers it.
  - Step 5, executable unresolvable: reachable through the stub. Hash mismatch: reachable by a config edit plus attach between the prompt and `y`.
  - Step 6, gone directory: reachable. On the confirm path it is checked only after the prompt (finding 2).
  - Step 7, failure message: reachable.
  - `validateButtonFlags` refusals: reachable only by a direct call; rows exist.
- Card 9:
  - Unresolvable executable: reachable through the stubbed `executablePath`.
  - `buttonClick` false: reachable.
  - The integration row's "neither strand-name variable set" assertion: **does not reach** the strip it claims to pin (finding 3).

**Findings:**

1. **MEDIUM, consistency.** `_lym/plan/04-geometry-button-context.md`, Tests, `TestGeometryOverRealHub` step.
   - Summary: the rows that set only the origin's `parent_worktree` to "another live pair" and to `gone-beam` never reach the parent-anchor or the empty-`Anchor` branch. `ParentOf` returns `""` (the prime) whenever the origin's `parent_branch` equals the main worktree's branch, before it reads `parent_worktree`. The existing step this extends changes only `ParentWorktree` and keeps the hubforge record's prime branch.
   - Fix: set `parent_branch` too, to a branch the prime has not checked out.
   - Evidence: `internal/hubgeom/parentof.go:30-46` (the `origin.ParentBranch == entry.Branch` early return); `internal/hubgeom/geometry_integration_test.go:94-100` (sets only `ParentWorktree = "other-task"`).
2. **LOW, design.** `_lym/plan/08-reed-button-verb.md`, `runButtonVia` steps 5–6.
   - Summary: the gone-run-directory guard (step 6) sits after the confirm branch (step 5). An unconfirmed click on a `confirm` entry whose directory is gone still raises the prompt, with `run-shell -c <gone dir>`, and refuses only after the operator answers `y`.
   - Fix: move the directory check before the prompt.
   - Evidence: card 8 Intent, steps 5 and 6 order.
3. **LOW, consistency.** `_lym/plan/09-button-click-binding.md`, Tests, `TestBindings_ButtonClickRunsTheVerbInTheRunDirectory`.
   - Summary: the "neither strand-name variable set, since the server's environment was stripped of them at spawn" assertion cannot fail because of the strip. `tmuxkit` already scrubs `agentname.StrandNameEnv` and `agentname.ParentEnv` from the test environment, so the server never inherits them, and `stripAgentNameEnv` is already pinned by a tier0 test.
   - Fix: drop that assertion, or state it as the run environment, not the strip.
   - Evidence: `internal/testkit/tmuxkit/tmuxkit.go:402`; `internal/reedengine/lifecycle_test.go:324-330`.

**Tool calls:**
1. `internal/reedengine/attach.go:85-110`, `internal/reedengine/lifecycle.go:322-345` and the `pinGeometryOptionsLocked` callers: to check whether card 6's unresolvable-shell guard can occur, since the earlier read lacked the shell resolution before the pins.
2. `internal/reedengine/paneoptions.go`, `setStrandPaneOptionsLocked` and the `s.Name` uses: to check card 7's empty-name branch, since its body was not read earlier.
3. `internal/hubgeom/geometry_integration_test.go`, the parent-step lines: to trace card 4's tier1 rows through `ParentOf`, since that test was not read earlier.
4. The strand-name env uses in reedengine tests and `tmuxkit`: to check whether card 9's env assertion can reach the strip, since the fixture's environment handling was not read earlier.
