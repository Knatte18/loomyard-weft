# board-field-drop: keep unknown board keys and bound the sweep retry

Batch 01 (`01-board-unknown-keys`) was the only batch and recorded `done` at head `e01121c305d242d377bf7f81553a39ad0dfed502`.
It covered the plan's three cards, each committed separately.

- Card 1: `boardengine` keeps unknown entry keys and unknown top-level keys as raw JSON across every write path, and `doc.go` states the retired-key rule.
- Card 2: `TestCLI` rows pin that `get`, `list-full` and the upsert, promote and merge envelopes print the kept keys, and `boardcli`'s `doc.go` says so.
- Card 3: `internal/lyxtemp` retries removal only on the not-empty symptom, under one injected-clock time budget per sweep.

The digest reported no deviations.
It carried three informational `scope-outside-plan` warnings: the test symbols `keptBoardBody` and `keptA` in `internal/boardengine/store_test.go`, and `taskKeys` in `internal/boardengine/task.go`, were touched outside the batch's own target glyphs.
The fork's own reply noted that card 1's gate path `./internal/boardengine_test` is not a directory, so it ran `./internal/boardengine` instead.
It also said it did not run other consumers of `boardengine.Task`, such as `cleanupcli` and `battencli`.

## Verify gate fixes

The plan-level verify failed on attempt 1: `internal/boardengine/boardtest` timed out, and the orchestrator reported that `TestConcurrentReadsDuringUpserts` spun at high CPU.
The fixer fork's reply names the cause as writer starvation, not an infinite loop.
Card 1's kept-keys decode ran under `Store.Load`'s shared lock and made each read several times slower, so the readers held the shared lock almost continuously and the writer's exclusive lock never got in.

The fixer committed `3db3b37ec` (a `fix:` commit in `internal/boardengine`):

- `Load` reads the raw bytes under the lock and decodes after releasing it, still wrapping a decode failure in `state.ErrDecode`.
- `loadedStoreFile.UnmarshalJSON` decodes each file once, collecting kept keys in a single byte pass; `Task.UnmarshalJSON` is unchanged.

The fork reported the test passing in about 0.04s and `lyx gate test` green for the three packages, untagged and with `--tags integration`.
It did not run the `go vet` tag runs, which the hook refused; the next plan-level verify covers them.
The fork also reported a transient merge conflict in `migrate.go` from its own stash and checkout while comparing; it restored `HEAD` and its edit and left the worktree clean.
