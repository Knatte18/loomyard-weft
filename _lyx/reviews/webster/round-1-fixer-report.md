# Webster round 1 — fixer report

All four findings were validated against the code and the plan, and each is fixed in its own commit.
`lyx gate test` is green over `./internal/orchengine`, `./internal/orchcli`, `./internal/boardcli` and `./cmd/lyx` after the fixes.

The gate then rejected the F2 commit's doc comment on `startAutoReload`, where one sentence ran across two lines ending in a comma.
`ba37d7aa7` splits it into two sentences, one per line; `lyx loom lint-comments` passes and the orchengine tests stay green.

## Fixed

- **F1** (`387fd56c2`): the watch verb now gets its index source from the new `orchCLI.watcherIndexSource`, which wraps `newIndexSource(c.strands, c.indexRunner)` and carries the recorded-binary rationale.
  `TestNewIndexSource` is renamed `TestWatcherIndexSource` and drives that method through `newTestCLI` with the CLI's `indexRunner` replaced by a recorder.
  Its rows cover the two behaviors card 14 asked `TestWatch` to pin: a strand with a recorded path runs the runner with that path, and a strand without one refuses with `lyx orch stop`.
  Reaching a role-file render through the watch verb itself needs a real `shuttleengine.Runner` over tmux, so this tier-1 test pins the construction instead, and orchengine's watcher tests already pin the render.
  If the watch verb swapped in an in-process render inside `watcherIndexSource`, a test would now fail.
- **F2** (`5d9930c2e`): `startAutoReload` clears `compactedAt` before going idle on a render failure, so the failure is reported once and the next turn end retries.
  `TestWatcher_AutoCompactionReloadGoesIdleWhenTheIndexSourceFails` gains a regression assertion: a later tick must not call the index source again.
  The assertion failed without the fix (2 calls, want 1).
- **F3** (`095ea7b1b`): the `board` group's `IndexNoteAnnotation` value is now `"— " + payloadLegend`, so the index line reads `…tasks and notes — A payload is one JSON object…`.
  The group `Long` still ends with the bare `payloadLegend`, so the grammar is still stated once.
  Checked by running `lyx help index`.
- **F4** (`a1c610b7e`): `Watcher.renderRoleFile` now words its own error by source, `command index unavailable: …` or `role stencil orch-template-role failed to render: …`, and the three render sites use it as the abort reason unchanged.
  `TestWatcher_FailingRenderAbortsWithoutClear` gains a `wantReason` prefix column, and the auto-reload failing-source test asserts the exact reason.
  Before, both tests asserted the stencil wording for an index failure, which was wrong.

## Disputed

- **F3, second half only** (the note printing under the `internal` audience): the plan requires this behavior, so it is not a defect against the fasit.
  Card 1 states the renderer contract without an audience condition: "a command carrying `IndexNoteAnnotation` gets the note appended to its line".
  The overview's decision `index-note-annotation` says "the renderer appends the value of a generic `IndexNoteAnnotation` to a group's line".
  Card 9 sets the note on the `board` group so "the index's `board` line ends with it".
  Under this rubric, `design` means the implementation does not match the plan, and here it does.
  Scoping the note by audience would change card 1's renderer contract, which is a plan change and not a diff fix.
  The run-on half of F3 is fixed above.

## Deferred

None.
