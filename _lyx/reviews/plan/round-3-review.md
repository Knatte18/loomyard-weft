---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/plan/13-bolt-push-seed-drop-retry.md
    summary: lyx fabric shortname is a mutating fabric verb with a record that pushes through Bolt.Push, yet card 13 lists it among recordless callers, so a seed drop at its push is only logged and missing from its envelope
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/08-prerun-seed-pull-first.md
    summary: cards 8 and 10 call PullThenCommitWritten without stating the record they pass, so a seed drop inside the pull-first of a recordless caller lands in a discarded record and is not logged, unlike card 13's rule for Bolt.Push
---

### [BLOCKING] `lyx fabric shortname` loses the push-path drop from its envelope

**Location:** `_lyx/plan/13-bolt-push-seed-drop-retry.md`, the Intent paragraph naming `Bolt.Push`'s callers.

**Issue:** Card 13 keeps `Bolt.Push` recording into a local `Mutations` and only logging the dropped SHAs, on the premise that its other callers ("board sync, `hubreconcile`, `configcli`'s hub-wide edit and `shortname`") have no mutation record.
The premise is false for `shortname`.
`runShortname` in `internal/fabriccli/shortname.go` builds `rec := fabricengine.NewMutations(l.HubPath)`, commits through `CommitShortname(..., rec)`, then calls `fabricengine.NewBolt(boardDir).Push(fabricengine.SyncOptions{})` and returns `okWithRecord(out, rec.Snapshot(), ...)` or `errWithRecord(out, rec.Snapshot(), err)`.
After card 13, that push can reset Bolt and drop seed commits, an observed state change, and the verb's envelope would not carry the `commits_dropped` entry.
`PATTERN-mutation-record` requires every mutating fabric verb to accumulate its record, and the decision record's constraint names the drop primitive as appending to the mutation record; this verb's envelope would silently omit a mutation it caused.
The other named callers hold up: `hubreconcile`, `configcli`'s hub-wide edit and `boardengine.Sync` build no `Mutations`, and `runReconcile`'s two pushes move to `PushRecorded` on this card.

**Fix:** Add `internal/fabriccli#runShortname` to card 13's `Edit:` list and switch its push to `PushRecorded` with the handler's `rec`, rewriting the `// Bolt.Push records nothing` comment there; drop `shortname` from the Intent's list of recordless callers, and have the `doc.go` sentence that names which pushes record the drop include it.

### [LOW] The pull-first drop of a recordless caller is recorded nowhere visible

**Location:** `_lyx/plan/08-prerun-seed-pull-first.md` and `_lyx/plan/10-hubreconcile-pull-first.md`, the `PullThenCommitWritten` call each describes.

**Issue:** Card 7 gives `PullThenCommitWritten` a `rec *Mutations` parameter into which `pullFirst`'s `dropSeedCommits` records `KindCommitsDropped`; it logs only the three skips.
Card 8's pre-run and card 10's `reconcileHubWide` have no envelope, and neither card states what record it passes or whether a drop is logged.
Card 13 decides for the same situation on the push path that a recordless caller logs the dropped SHAs at Info "so a caller without an envelope still sees the drop in the log"; on the pull path the same drop by the same primitive would land in a discarded record with no log line, so a reset of Bolt by the pre-run is invisible.

**Fix:** State on card 7 that `PullThenCommitWritten` logs the dropped SHAs at Info, as card 13's `Bolt.Push` does, or state on cards 8 and 10 the local record each passes and that each logs its `KindCommitsDropped` entries.

## Focus verdicts

- Push-path drop recording (cards 10, 11, 13): `runReconcile`'s hub-config and warp-binding pushes record through `PushRecorded`, and card 13 orders after card 11, which edits the same handler.
  `hubreconcile` builds no `Mutations` and only logs, which card 13 states.
  The defect is the `shortname` caller, F1.
- Card 7's test write surface: `internal/fabricengine/export_test.go` is `package fabricengine` and is on card 7's `Edit:` list, which is what lets the `package fabricengine_test` integration file reach `dropSeedCommits` and `boardDropRequest`; `destructiveguard_test.go`, `livestate_mutationoracle_test.go` and the budget file are listed too.
  Both fixtures state their construction through `hubforge`: the main one over a `hubforge` hub with a second clone of `RecordsBare`, the foreign `_board` as a linked worktree of the same hub's records repo at `<t.TempDir()>/elsewhere/_board`, whose parent holds no `WeftWarpSlug` sibling, so `isRegisteredLinkedWorktreeIn` finds no owner. Not a defect.
- Card 8's label source: the current `seedSubtree`'s `label` parameter is the subtree name (`"stencils"`, `"specs"`), so `SeedCommitMessage(label, BinaryLabel())` puts the subtree first as card 6's signature requires, and the label comes from `BinaryLabel()` of the test process, the same source the card's assertion computes; the fixed identity drives only the writer keys. Not a defect.
- Cherry-pick replay of an already-carried change: card 5 runs `git cherry-pick --empty=drop`, states the commit is dropped with exit 0, and `TestCherryPick` has the row where HEAD stays unmoved; card 7 states the same outcome for `dropSeedCommits`' replay. Installed git 2.53 lists `--empty (stop|drop|keep)` for `cherry-pick`. Not a defect.
