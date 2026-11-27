```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens, covering what the diff's changes force outside the lines it touches.

**Per change: the set found, covered or not**

| Change | Set | State |
|---|---|---|
| `PATTERN-pair-remote` declared line | `PATTERN.md`, background opening | covered, verbatim |
| Detached spawns start `lym loom pusher` | callers of `Spawn*`; `main.go` `detachedChild` | covered |
| | tender lift's deferred-push payment | **not covered** (F2) |
| | `webster reset`'s owed record | **not covered** (F3) |
| | help text and comments describing the old child | **not covered** (F5) |
| New `--board` flag read by `detachedChild` | other commands declaring `--board` (none found) | covered |
| Board spawn replaces `board sync --deferrable` | `boardengine/doc.go` | covered; `board.go`, `hubreconcile/doc.go`, `tendercli/doc.go` not (F5) |
| Pusher per target (board and landing locks) | `runpusher/doc.go` | covered; `lymdirs`, `PATTERN-run-fsm`, `docs/overview.md` not (F5) |
| Finalize no longer pushes; seams removed | `envkit`, `loomcli` wiring | covered; `Fabric.PushBranch`, `LocalOnlyCommits` orphaned (F1) |
| Landing target gets a push (card 11) | `loomcli/doc.go` | **not covered** (F5) |
| Teardown pushes nothing; `teardownPushError` removed | `battenshed` docs; push sentinels | sentinels **not covered** (F4) |
| `PushUntilLanded` deleted | wake-up file list, `PATTERN-daemon-wakeups` | covered |
| Network re-step without cap | `PATTERN-shed-driver`, reflection template, anomaly doc | covered |
| `held_back` class | `FailedPushWayForward`, `pushStep`, owed-class reads | covered |
| New fabric keys | template, categories, `configreg` test | covered |
| Edit directives, reflection template | both engines | covered, no rule dropped |
| New `.lym` records (ledger, wait record, merge-in base) | `lymdirs` names; `PATTERN-lym-files` cache list | list not updated (F6) |

**Findings**

**F1. BLOCKING, design.** `internal/fabricengine/fabric.go:192` and `internal/fabricengine/parenttips.go:73`
- `Fabric.PushBranch` and `Fabric.LocalOnlyCommits` are now called only by tests, which breaks `PATTERN-test-only-code`.
- The diff removed their production callers: `parentMerger.PushBranch` in `landingshed/finalize.go`, and the `ParentCommitsOriginLacks` seam in `loomcli/landingdeps.go`.
- Evidence: a word grep over `internal` and `cmd` finds only the definitions, a line in `fabricengine/doc.go:291` and `_test.go` calls. The dead-code guard runs only under `upkeep`, so no task gate catches this.

**F2. MEDIUM, design.** `internal/tenderengine/resume.go` (`pairsToPush`, with `deps.PushDeferred` wired in `internal/tendercli/deps.go:141`)
- A hub lift now pushes no pair that a pause held back.
- `pairsToPush` skips every pair for which `PushDeferred` is false. That reads the pending flag written only by a detached `lym fabric push` child deferring under a pause.
- Nothing spawns that child any more: `pushSpawnCommand` (`internal/fabricengine/spawn.go`) now builds the pusher line. The pusher exits on a pause (`errPauseCovers`, `internal/runpusher/run.go`) and leaves no flag.
- The dobby restarts only the hub's own targets and batten runs' pairs. A hand-made pair, or any pair with no batten run, therefore stays unpushed after the lift until its next commit.
- `tendercli/cli.go:213` and the `spawnOwed` doc still promise the push.
- Related, not traced further: the prime and landing pushers carry no pause flag and no `PauseCovers`. `lym fabric sync` in the prime previously spawned a deferrable child (`--pause-slug=`).

**F3. MEDIUM, design.** `internal/webstercli/reset.go:184` (through `fabricSync`, `internal/webstercli/sync.go:25`)
- D1 lists "webster reset's records push, whose owed record holds the run at its transition" among the pushes that keep their own behaviour.
- That push is `Fabric.Commit`'s spawn, which now starts the pusher. The pusher pushes with `OwedUnrecorded`, so a transport failure writes no owed record and the stated hold is gone.
- The diff rewrote `pattern/PATTERN-run-fsm.md` to drop the hold. `pattern/PATTERN-pair-remote.md` lists "`webster reset`'s move of the code branch" as the synchronous push instead of the records push.
- No `Pattern-Departure:` trailer or added entry covers this. Plan card 7 does route every detached spawn to the pusher, so the handler should weigh card against D1.

**F4. LOW, design.** `internal/battenshed/teardown.go:205-208` and `internal/battenshed/deps.go:361-377`
- `ErrPushEntryEnded` and `ErrPushHandedOver` have no producer left. `teardownPushError` was deleted, and the new `Push` only marks the ledger and starts the pusher (`internal/battencli/wire.go`).
- The two branches and the `entryEnded` path are now unreachable in production. The `TeardownDeps.Push` doc still describes them.

**F5. LOW, consistency.** Descriptions the change makes false:
- `internal/loomcli/doc.go:233`: "the landing target has no push yet", though card 11 added it. This line is new in the diff.
- `internal/lymdirs/dirs.go:76` and `internal/lymdirs/doc.go:10`: "one lock per records worktree". `board.lock` and `landing.lock` now sit there too.
- `pattern/PATTERN-run-fsm.md:27`: "one per records worktree".
- `docs/overview.md:341`: "the pusher of a records worktree".
- `internal/boardengine/board.go:359`, `internal/hubreconcile/doc.go:19` and `internal/tendercli/doc.go:62`: the detached `lym board sync` that no write spawns now.
- `internal/fabriccli/weft_verbs.go:393` (user-facing help for `fabric sync`): a detached child pushes at once, "coalesced under fabric's push lock".
- `internal/fabriccli/fabric.go:129` (help for `add`): the child pushes "as `lym fabric sync` does". It now starts the interval-paced pusher.
- `internal/fabriccli/roleguard.go:333`: "The detached push child every commit spawns ... carries the hidden path flags".

**F6. NIT, consistency.** `pattern/PATTERN-lym-files.md` (cache list under "Durable and ephemeral state")
- The list names everything run-related under `.lym` and omits the push ledger, the drive-wait record and the merge-in base file.

**Tool calls**
1. Grep for `--board` flag declarations, orphaned symbols and the reset push: callers beyond the manifest.
2. Grep for stale prose about the old push behaviour: docs and comments outside the diff.
3. A caller grep (it errored) and `tenderengine/resume.go` `spawnOwed`/`pairsToPush`: the lift path was not in the manifest.
4. Word grep for `PushBranch`, `LocalOnlyCommits`, `OpenParent` and `HoldsWork`: the retry of the errored caller search.
5. Reset push grep and the help or doc lines in `fabric.go`, `weft_verbs.go`, `hubreconcile/doc.go`, `board.go` and `roleguard.go`: full text of the grep hits.
6. Grep of `webstercli` for how reset pushes its records: not in the manifest.
