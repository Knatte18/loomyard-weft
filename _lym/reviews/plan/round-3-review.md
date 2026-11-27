---
verdict: APPROVED
prior:
  - id: R1
    status: resolved
  - id: R2
    status: resolved
  - id: R3
    status: resolved
---

The three round-2 fixes hold, and I found no new `BLOCKING` defect on a changed hunk.

- R1: `reportUpdateFrom` takes `tilingOutput`, cards 9 and 11 state the landed desktop and output per path,
  and the mode `off` row asserts `LastDesktop`, `Home` and `Output`.
- R2: the apply step is its own `tier3` test with no skip or flag, and both `-run` lines are named.
- R3: `Spawn` registers only a non-prime child under a non-empty `pauserecord.RunProcessEnv`,
  the Shared Decision says the same, and every `TestPlaceWindowChild` row sets the variable.

## Focus questions

- Report update: after an apply the places file takes the plan's target and `tilingOutput`;
  under `off` it takes the placed window's snapshot desktop and output, and keeps both desktops when there is no single one.
  Every `ReportUpdate` value that card 11 lists has an input.
  The `off` path has a named assertion.
  The after-apply path has none for the placed entry's `LastDesktop`, `Home` or `Output` at the flow level,
  and `TestUpdatePlaces` builds its `ReportUpdate` by hand, so the join of apply report to snapshot is unasserted for them.
  One D8 case lacks an input; see the first note below.
- Contract test: neither test skips, both are `tier3`, and the `tier3` command in `gate.yaml` takes `{args}`,
  so `-- -run TestContract_TheRealKWinAnswersAsRecorded` runs the read-only steps alone and moves no window.
- Registration: `shedverbs`' `run` and `step` and `beam dobby` carry the run-process annotation,
  and reed strips the mark when it boots a server, so the rule matches D10.
  The readers card 12 names behave as it says.
  `tenderengine`'s process report and `pairteardown.EndSubjectProcesses` also read records without picking a role;
  both stay correct, since the first reads a record of a present worktree as no orphan and the second should end the child.
- `TestPlaceWindowChild`: every row sets the variable, and the restricted `PATH` makes the run fail before any desktop.
  The waiting window command has two gaps; see the notes.

## Notes below the incremental bar

Each is below `BLOCKING` or on unchanged text, so none is a finding.

- A deleted home desktop is never replaced.
  Card 10 places such a window by mode (D8), but card 9's `updatePlaces` gives a new home only when the entry had none
  or the window landed on its home, and `ReportUpdate` carries no desktop list to tell that the home is gone.
- `window_command` can run before the window exists.
  Card 11 settles on a failed probe in the first snapshot, which may not show the window yet, and then runs `afterPlace`;
  card 12's resolution-failure path does the same.
  D12 drops `window_wait_ms` because the child has waited for the window itself.
- Cards 9 and 11 give no desktop for another hub window that sits on several desktops or on all,
  so the hand-move rule's input is undefined for it, and no test row covers it.
- `Anchor` answers an empty desktop with `true` for a settled parent with no live window and no last desktop.
- Card 12's window command must wait while `PATH` holds only `sh` and `git`, so it needs `sleep` by absolute path.
- The `PrimeWindow` row does not say its command is still waiting, or that its release file is removed,
  when it asserts that no `RolePlacement` record exists.
  A finished child's record is pruned, so the assertion can pass vacuously.
  The record to read is the one under the prime's worktree name, which is what `Spawn` would register.
- `TestContract_TheRealKWinAppliesAsRecorded`'s doc comment is told to name the moved throwaway window only.
  The apply also rebuilds the grid of the chosen desktop and releases the windows tiled there,
  and the card does not say who opens and closes the throwaway window.
- `tmuxkit.Main` points `XDG_STATE_HOME` at a temp directory, so both contract tests take a script lock
  that no running placement shares; the operator should run them while no placement is in flight.
- `CurrentOrder` matches a clamped window by its frame's top-left corner with no stated tolerance,
  and no test runs it over the apply recordings from Neve's fractionally scaled screens.
- Card 8's `Query` reads `kwinquery.json` under the script lock, while `run` takes that lock itself and card 8 names no hook.

## Constraint and record

The record asks for the placement rules under tier-1 tests, and the plan puts them at `tier0`.
`PATTERN-test-speed` puts a behavior at the lowest tier its real modules run in, so the constraint wins.
