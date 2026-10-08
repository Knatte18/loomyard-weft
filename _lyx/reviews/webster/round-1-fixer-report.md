# Webster round 1 — fixer report

All five findings were validated against the code and the plan, and all five are fixed.
Verify: `go build ./...`, `go vet` under the default, `integration`, `tmux` and `llm` tags, `go test ./...`, `go test -tags integration ./...`, `go test -tags "integration tmux" ./internal/reedengine/` and `lyx loom lint-comments --base <merge-base>` all pass on the final commit.

## Fixed

### F1 [BLOCKING] Process-costing reads run on every poll check

Confirmed: every `battencli/wire.go` seam reached `taskWorktreeLocation`, whose `lyxcwd.ResolveWorktree` runs `git rev-parse`, so `ReadDecision`, `StopReport` and `AttachDir` spawned git on every 2 s check.

- `wire.go`: a new `taskWorktreeLocator` keeps the first successful resolution for the wiring's lifetime; later calls only stat the pair, so an absent pair is still refused by name.
  The stat and refusal moved into `presentTaskWorktreePath`, which `taskWorktreeLocation` also uses.
  Every seam in `wire` locates the task worktree through that one locator; `readStopReport` and `markBattenWatched` now take the resolved location.
- `battenshed/doc.go` and the `NoticeProbe` doc in `deps.go`: the rate-of-reads sentence now says the notice step's own file reads run per check, and lists what is probe-gated (driver, review and agent-activity reads, forced decode, not-parked retry, watched-marker refresh, a notice's delivery).
- Coverage gap closed: `stepTaskWorktreeSeams_RunGitOnlyUntilTheFirstResolution` in `lifecycle_integration_test.go`. Over a real pair, once `AttachDir` has resolved, `AttachDir`, `ReadDecision` and `StopReport` keep answering with the pair's `.git` hidden, so git cannot resolve it. With the pair moved off disk, the call is refused by name.

Commits `f5f2a90ed`, `6476ee194`.

### F2 [MEDIUM] Down reads reed state to find its reap roots

Confirmed: `sessionReapRootsLocked` went through `listStoredStrandPanes`, which calls `LoadState`. With a corrupt `reed.json`, the reap was skipped, and a batten window was never reaped.

- `overlay.go`: a new `TmuxCmd.listSessionPanes` (`list-panes -s -t =<session>`) shares the pane format with `listPanes`.
  `reapSessionPanes` takes the session name and lists every window, and both session reaps use it: `Engine.Down` (through `sessionReapRootsLocked`) and the engine-less `ReapSession`. Neither reads reed state, and both cover a second window's process tree.
  This departs from one clause of card 9 ("the session-kill reap keeps the current-window fallback"): kill-session ends every window, so the session-wide listing is a strict superset of the current-window one, and the reviewer's fix asked for it.
- `Down`'s doc comment ("loads no state") is true again. The `sessionReapRootsLocked` doc and `doc.go`'s strand-window paragraph now describe the session-wide reap.
- `faketmux_test.go`: the unscripted session-wide listing answers in the shape its format asks for.
  `TestReapSession_CallOrdering` expects the session-wide verb, and `TestReapSessionPanes` asserts the `-s -t =myworktree` target.
- Coverage gap closed: `TestDown_ListsEveryWindowsPanesOverACorruptState`. I saw it fail against the old `listStoredStrandPanes` line before the fix.
  The reedengine row in `cmd/lyx/testdata/test-budget.yaml` goes from 215 to 216 for it.

Commits `0ae17155e`, `3b7f529bb`.

### F3 [LOW] Wait reason still says the row sleeps before the next bounce

The running arm's reasons now read `inner shed run still running; checking it every <poll>` and `inner shed run waiting: <note>; checking it every <poll>`.
`TestInnerRun_RunningArmReviewWaitNote` was corrected to the new text. Commit `dc959cb2c`.

### F4 [LOW] Watched stencil names who announces the stops

The opening sentence "Something else watches this run and tells the parent about every stop, so you message…" is now "Message the parent session that the directive above names in exactly three cases…", so it no longer says who announces stops or claims that every stop is announced.
The closing clause "those stops reach the parent by other means" stays; see Disputed. The existing claims in `contracts/stencils/claims_test.go` still hold. Commit `644796af5`.

### F5 [NIT] Changed comments keep column wrapping; pattern text is stale

- `panebin.go` header: one sentence per line, now naming both chokepoints (`launchStrandLocked` for strand panes, `OpenWindow` for detached windows).
- `reedengine/doc.go`: the sentences this task touched are now one per line: the `liveBoxLocked` sentence, the chained-attach bullet's first sentence and the geometry-pins bullet's first sentence.
- `pattern/PATTERN-pane-binary-resolution.md`: the first bullet names both call sites.

Commit `dfbe17745`.

## Disputed

- F4, the closing clause only: card 3 (`_lyx/plan/03-driver-notify-rule.md`, the `shed-template-driver-notify-watched` bullet) gives the watched rule's wording itself: "It never messages the parent that the run stopped, halted or finished and sends no generic escalation line, … because those stops reach the parent by other means."
  "By other means" names no announcer, so it does not break the card's "says nothing of who announces a stop", and removing it would depart from the card.
  The rest of F4, the opening sentence, is fixed above.

## Deferred

None.
