# Webster-Review round 1 — fixer report

All 14 findings are fixed, one commit per finding on the warp branch, with nothing pushed.
F1 and F2 share one commit because both rewrite the same table rows in `contracts/specs/refusal-spec.md`.
F11 is split into a production-comment commit and a test-comment commit.

## Fixed

- **F1** (`7f01c4d35`): `refusal-spec.md` gains rows for every refusal the review listed.
  The webster rows are report already present, recovery batch at record-batch, recover-batch over an OK report, merge in progress, HEAD moved past the report, unqualified merge, no run in progress, plan-dir override, recovery state vanished, outcome malformed, run-exit check failed, and Webster row paused out of band.
  The shed rows are seed missing and status watch as JSON.
  The loom rows are seed missing, status file missing (status and approve) and start: driver not parked.
  While adding the HEAD-moved row, the non-merge-movement refusal in `internal/websterengine/gitwrap.go` turned out to name no way forward at all.
  It now ends `way forward: move HEAD back to the report's head_sha <sha> with git, then re-run this verb`, which is the step its existing test already takes, and that test now asserts the clause.
- **F2** (`7f01c4d35`): cells now quote the reworded messages.
  The run-exit and parent-write rows say "in the task worktree", and the preflight cell gives the three per-check clauses.
  The fingerprint trigger drops record-batch, the re-baseline-persist row is split into its bracket-verb and validate forms, and the goto-target and budget cells match the messages.
  The spec's intro now says a message that already named its way forward before the audit keeps its own wording, which its cell quotes.
- **F3** (`0e1f112dd`): recover-batch's reed-boot and vanished-state refusals name `lyx webster recover-batch NN` with a bare number.
  The reaching test asserts the exact runnable command.
- **F4** (`90862faa3`): rebaseline's fabric-sync refusal appends `fabricSyncWayForward`.
  A new test, `TestRebaselineCmd_FabricSyncFailureWayForward`, reaches the refusal and checks that the restamped state was saved.
- **F5** (`958225641`): Master's ladder gains a recover-batch `{"batch_failed": true}` rung.
  It treats a rejected recovery like a terminal stuck or dead one: write `outcome: stuck` and stop, never recover-batch again.
  `TestMasterTemplate_TeachesBatchFailedAndReportArchivedRungs` pins the rung.
- **F6** (`f5e159792`): goto's missing-status refusal shares step's seed way forward through a new `missingStatusWayForward` const.
  The unknown-target and missing-`--to` refusals end `way forward: re-run goto with --to naming one of: …`, and goto's busy refusal reuses `busyWayForward`.
  The tests assert the clauses, and a new `TestGoto_MissingStatusFileNamesSeeding` takes the missing-status way forward.
- **F7** (`cf122442c`): both fingerprint-mismatch refusals end in a shared `fingerprintMismatchWayForward` clause.
  The begin-batch and run tests assert it through `requireWayForward`.
- **F8** (`4210f4487`): websterengine's doc now names `rerunCardVerifies`, states the real correctness classes, says begin-batch and run for the fingerprint refusals, and states rebaseline's exact card-set condition.
- **F9** (`fb3f8155b`): shedengine's doc describes the prefixed budget reason and the goto episode boundary, and so does loom-status-spec's budget rule.
  The four "four generic verbs" comments name no count.
- **F10** (`788fb8b0c`): with no recorded StartSHA, the rebaseline refusal names "the run's start commit" with no blank slot.
  A new test pins this.
- **F11** (`0f385a91b`, `224d017f2`): every new or changed Go comment the diff hard-wrapped mid-sentence is rewrapped to one sentence per line, broken only at clause boundaries.
  A re-scan of the diff finds none left.
  Untouched legacy lines were left as they are.
- **F12** (`bc87ea61a`): `TestFabricSyncWayForward_NextSyncCommitsSavedState` builds a real `hubforge` hub.
  It saves state, fails a sync, then shows that the next `fabricSync` commits `_lyx/webster/state.json` into the weft.
  The per-verb tests' comments now claim only what those tests show.
- **F13** (`af21975f1`): batten's parent `Long` example uses `--to Worktree-Teardown`.
- **F14** (`c9dc0f6b8`): `failBatch` uses `slices.Contains` and `containsString` is gone.
  The engine `rebaseline.go` now has a file header.

## Verification

`go build ./...`, `go vet ./...` and `go test ./...` pass.
So do `go test -tags integration ./...` and `go vet -tags smoke ./...`.

## Deferred

Nothing.
