---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/13-bolt-push-seed-drop-retry.md:21
    summary: Bolt.Push's drop appends to a throwaway local Mutations, so lyx fabric reconcile, a mutating fabric verb whose envelope carries mutations and whose handler card 11 edits, performs a reset of _board its record never shows
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/07-bolt-pull-first-seed-drop.md:64
    summary: TestBoardDropRequest_RefusesForeignBoard sits in a package fabricengine_test file but drives the unexported dropSeedCommits, and card 7 does not list internal/fabricengine/export_test.go, the package's shim for exactly that
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/07-bolt-pull-first-seed-drop.md:64
    summary: the foreign _board fixture of TestBoardDropRequest_RefusesForeignBoard is the one card 7 geometry whose construction is unstated, so nothing keeps it off a hand-assembled repo
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/08-prerun-seed-pull-first.md:30
    summary: the scenario test passes a fixed identity yet asserts each seed subject carries the label's revision, while BinaryLabel reads buildvcs.Running() of the unstamped test binary and ignores the passed identity
  - id: F5
    severity: LOW
    class: design
    location: _lyx/plan/07-bolt-pull-first-seed-drop.md:49
    summary: the drop's cherry-pick replay treats a non-seed commit that upstream already carries as a conflict and skips diverged, where the pull --rebase it replaces drops such a commit silently
---

### [BLOCKING] The push-path drop is invisible to `lyx fabric reconcile`'s mutation record

**Location:** `_lyx/plan/13-bolt-push-seed-drop-retry.md:21`

**Issue:** Card 13 has `Bolt.Push` pass `dropSeedCommits` a local `Mutations` and only log the dropped SHAs at Info, on the premise that every `Bolt.Push` caller is outside this task's scope.
That premise is false for two of them: card 10 edits `hubreconcile`'s push path and card 11 edits `runReconcile`, whose hub-config push (`internal/fabriccli/fabric.go:774`) and warp-binding push (`fabric.go:842`) both go through `Bolt.Push`.
`lyx fabric reconcile` is a mutating fabric verb that threads one `rec` through everything it does and emits it under the `mutations` envelope key.
With card 13, a rejected push in that verb can reset `_board` to upstream and replay commits, a destructive, observed change, and the verb's own record never carries the `KindCommitsDropped` entry.
`PATTERN-mutation-record`, a constraint the decision record names for the drop primitive, requires the verb to accumulate its mutations in its record, and D5 says the primitive "appends to the mutation record", the verb's, not a scratch one.
Because card 7 makes `KindCommitsDropped` a git-state kind (`false` in `manifestObservableKind`), the truthfulness oracle cannot catch the omission either.
The pre-run precedent card 13 cites does not transfer: the pre-run has no envelope, while `fabric reconcile` has one.

**Fix:** Add a recording sibling on `Bolt`, for example `Bolt.PushRecorded(opts SyncOptions, rec *Mutations) error`, holding card 13's retry and passing `rec` to `dropSeedCommits`; `Bolt.Push` delegates to it with the local recorder and the Info log for callers that have no envelope.
Card 11 lists the sibling under `Uses:` and switches both `runReconcile` pushes to it with the handler's `rec`, and its scenario test asserts the drop shows in the envelope's `mutations`; card 13's Intent states which callers record the drop and which only log it.

### [MEDIUM] Card 7's foreign-board test cannot reach `dropSeedCommits` without the export shim

**Location:** `_lyx/plan/07-bolt-pull-first-seed-drop.md:64`

**Issue:** Card 7 places its tests in "the new `package fabricengine_test` integration file", and `TestBoardDropRequest_RefusesForeignBoard` "gives `dropSeedCommits` a `_board` worktree".
`dropSeedCommits` and `boardDropRequest` are unexported, so an external test package cannot call them.
The package's established route is `internal/fabricengine/export_test.go`, whose header says it re-exports "the destructive gate's unexported executors and ownership predicates" for `fabricengine_test` files; card 7's `Edit:` list carries `destructiveguard_test.go` and `livestate_mutationoracle_test.go` but not that file, so the implementer cannot add the shim inside the card's write surface.

**Fix:** Add `internal/fabricengine/export_test.go` to card 7's `Edit:` list, and state in the Intent the re-export the test drives (the drop primitive with `boardDropRequest`).

### [LOW] The foreign `_board` fixture's construction is unstated

**Location:** `_lyx/plan/07-bolt-pull-first-seed-drop.md:64`

**Issue:** Every other card 7 test names its geometry, a `hubforge` hub whose upstream a second clone of `RecordsBare` moves, as the decision record's constraint "Bolt tests build on `hubforge`" requires.
The foreign-board test names only the property its fixture must have, a `_board` no weft sibling of its parent registers, and not how it is built.
A `_board` init'ed or cloned by hand in a temp dir would satisfy the sentence and break `PATTERN-hubforge-fixtures`' "no hub is hand-assembled" for the one fixture that exercises the new ownership edge.

**Fix:** State the construction in the Intent, for example a linked worktree of the `hubforge` hub's weft repo added at `<tempdir>/<other>/_board`, so the target is a real registered `_board` whose parent holds no weft sibling.

### [LOW] The scenario's label assertion and the fixed identity disagree

**Location:** `_lyx/plan/08-prerun-seed-pull-first.md:30`

**Issue:** Card 8 re-signs `seedStencilsAt` to take the identity "so the integration test can drive a production mode and a chosen identity without a stamped test binary", and the scenario's first step asserts "each subject carrying the label's revision, channel and executable".
The subject comes from `SeedCommitMessage(label, BinaryLabel())`, and card 6's `BinaryLabel` reads `buildvcs.Running()`, `buildinfo.Channel` and `os.Executable()`, never the passed identity.
Under the unstamped test binary the label is `unknown unstamped <test binary>`, so a reader of the step cannot tell whether it asserts the fixed identity's revision, which would fail, or the running binary's.

**Fix:** Say the step asserts the subject equals `SeedCommitMessage(<subtree>, BinaryLabel())` from the test process, or pass the label into `seedStencilsAt` beside the identity so the fixed identity drives both the writer keys and the subject.

### [LOW] An already-applied non-seed commit fails the replay

**Location:** `_lyx/plan/07-bolt-pull-first-seed-drop.md:49`

**Issue:** `dropSeedCommits` replays each non-seed ahead commit through card 5's `CherryPick`, a plain `git cherry-pick <sha>`.
When upstream already carries that commit's change, git stops the pick as empty and exits non-zero, which card 7 handles as a replay conflict: abort, restore, `BoltSkipDiverged`.
`pull --rebase`, which the drop stands in front of, drops such a commit and lands.
The outcome is safe, since Bolt is restored and the Warn names `git pull --rebase`, but neither card states the case, and no test row reaches it.

**Fix:** State the empty-pick rule in card 5's `CherryPick` Intent, either skipped as `pull --rebase` skips it or reported as a conflict on purpose, and add the matching row to `TestCherryPick` or card 7's replay subtests.

## Verdicts on the focus directives

- Both subtrees due in one pre-run: not a defect.
  Card 8 makes one `PullThenCommitWritten` call carrying the stencils write then the specs write, and card 7 runs `pullFirst` once before every write, so the drop never sees a seed commit the same pass made.
  The scenario's first step covers both subtrees due together and asserts both seed commits remain on Bolt after the pass.
- Card 7 test geometry: the pull-first and push-retry tests build on a `hubforge` hub, as `PATTERN-hubforge-fixtures` requires; the one unstated fixture is F3, and the unreachable unexported call is F2.
- Card 7 drop gate: not a defect.
  `ownedBoardWorktree` binds the target beyond its base name: a sibling directory whose name `WeftWarpSlug` admits must register it through `isRegisteredLinkedWorktreeIn`, which matches the hub's real layout, where `_board` is a linked worktree of the `-weft` repo, and the Intent states that bound and force always false.
  A dirty tracked file on a touched path refuses through `dirtyTrackedOn` and returns `BoltSkipDirty`, whose detail names the dirty paths, `lyx board sync` and a re-run; a dirty file elsewhere is carried across, and a subtest covers each case.
- Card 7 and 13 recorder: `pullFirst` appends to its caller's `rec`, but `Bolt.Push` appends to a throwaway recorder; F1.
- Card 12 `IsNoServer`: not a defect.
  The Intent names both tmux forms, `no server running on` and `error connecting to … (No such file or directory)`, and `TestListSessions` in the untagged `overlay_test.go` gains a row for each plus an unrelated failure, so `go test ./...` runs them.
