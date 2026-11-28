# Card 4 — origin-reads-seams

**Edit:**
- `internal/fabricengine#OriginRunStatus` -> `func OriginRunStatus(prime *lymcwd.Location, slug string) (status []byte, found bool, err error)`
- `internal/fabricengine#originStatusAtTip`
- `internal/fabricengine#ReadOriginFor`
- `internal/fabricengine#MarkBeam`
- `internal/fabricengine#Fabric.RemoteOnlyCommits`
- `internal/fabricengine_test#TestOriginCodeTip`
- `internal/fabricengine#readCodeTipAt`
- `internal/fabricengine#OriginCodeTip`
- `internal/pairteardown#hubRunReader`
- `internal/fabricengine_test#TestOriginRunReads`

**Create:**
- `plan:internal/fabricengine#refuseUnsuppliedSeams` -> `func refuseUnsuppliedSeams(needed ...fabriccore.RequiredSeam) error`
- `plan:internal/fabriccli_test#TestInstallRequiredSeams` -> `func TestInstallRequiredSeams(t *testing.T)`
- `plan:internal/fabricengine#originStatusBytesAtTip` -> `func originStatusBytesAtTip(prime *lymcwd.Location, recordsRoot, tip, slug string) ([]byte, bool, error)`
- `plan:internal/fabriccore#MarkBeamVerdict` -> `func MarkBeamVerdict(recordFound, alreadyMarked, recordsWorktreePresent bool) (mark, missingWorktree, missingRecord bool)`
- `plan:internal/fabriccore#ReadOriginForVerdict` -> `func ReadOriginForVerdict(recordPresent, lockDirPresent bool) (read, missingLockDir bool)`
- `plan:internal/fabriccore#RemoteOnly` -> `func RemoteOnly(hasOrigin bool, tips map[string]string, branch string) (tip string, read bool)`
- `plan:internal/fabriccore_test#TestOriginRecordRules` -> `func TestOriginRecordRules(t *testing.T)`

**Uses:**
- `internal/fabricengine#SetRunRecordPaths`
- `internal/fabricengine#SetStatusCodec`
- `internal/fabriccore#RefuseUnsupplied`
- `internal/fabriccli#InstallRequiredSeams`
- `internal/pairteardown/doc.go`

**Intent:**
`internal/fabricengine/originreads.go` stops importing `internal/shedengine` and `internal/shedrun` (D13): its status reader returns bytes its caller decodes.

- `refuseUnsuppliedSeams` reads `fabriccore.FieldSeams` into an empty snapshot through `snapshotReader{}` in `PhasePreLock` and returns `fabriccore.RefuseUnsupplied`'s refusal, or nil.
  It is the pre-lock refusal that `OriginRunStatus` and `OriginCodeTip` (this card), `coalescePushSides` (card 7) and `PushPairAnchored` (card 8) run before they read or act;
  the input that reaches it is a process with a required seam unset.
  `ReconcileOrigin` refuses through its core rule (card 9).
  `ReadUpstreamTakeover` meets it through `originStatusAtTip` inside the push verbs' records retry (`takeoverStop` and `primePushTakeovers` in `retryWeftPush`), which those verbs' refusal covers.
  The handover readers `OriginParentRunStatus` and `OriginBattenStatus`, and the landing hand-over's own `ReadUpstreamTakeover` call (`internal/loomcli`'s `runHandOver`), meet it only through `originStatusBytesAtTip`;
  their pre-action refusal is left to part 4.
- `OriginRunStatus` refuses first when `SeamRunRecordPaths` is unsupplied, then reads as today and returns the status record's bytes at the tip, with found false when the ref or the record is absent.
  `originStatusBytesAtTip` is today's read of the record at the tip: the anchor, the record's tree path from `RunRecordPaths.TreeStatusRel` in place of `shedrun.TreeRunFileRels`, then the bytes unparsed.
  It returns `refuseUnsuppliedSeams(SeamRunRecordPaths)`'s refusal when the seam is unset, so no caller reads through a nil seam.
- `originStatusAtTip` keeps its signature and its `shedengine.Status` result for its part-4 callers, `OriginParentRunStatus` and `OriginBattenStatus` (`handover.go`) and `ReadUpstreamTakeover` (`takeover.go`).
  It moves to `handover.go`, which already imports `internal/shedengine`, as a decode with `shedengine.ParseStatusTolerant` over `originStatusBytesAtTip`.
  Only the function moves and handover.go's own code is untouched: it is the one edit of another concern's file this part makes, so `originreads.go` can drop its shed imports (D13, D9).
- The origin record's and the remote-only read's rules move to the core (`origin` and `remoteonly` of the record's Goal).
  `MarkBeamVerdict` is `MarkBeam`'s choice: a missing records worktree, a missing record, a record already marked (no write, false), or a mark.
  `ReadOriginForVerdict` is `ReadOriginFor`'s: no record reads not found, and a record without its lock directory is the error naming `lym fabric reconcile`.
  `RemoteOnly` is `RemoteOnlyCommits`' early returns: no origin remote, and no remote branch of the checkout's name, read empty; otherwise the fetch and the commit list run.
  The engine asks it first with `hasOrigin` alone and nil tips, and a false answer ends with no further read, as today's origin-URL check returns before any other read.
  Nil tips mean origin's tips are not read yet, so with `hasOrigin` true that first call answers `read` true; a non-nil map without the branch is no such branch and answers false.
  Only after a true first answer does the engine read the branch, the head and origin's tips and ask it again.
  The engine keeps the stats, the reads, the write, the commit and every message word for word.
- `OriginCodeTip` refuses first when `SeamRunRecordPaths` or `SeamStatusCodec` is unsupplied.
  `readCodeTipAt` reads the machine of the last transition's status through `StatusCodec.LastMachine` over the bytes, in place of parsing `shedengine.Status` itself.
  Its result is unchanged (D5).
- `pairteardown.hubRunReader`'s `childOnOrigin` closure decodes `OriginRunStatus`'s bytes with `shedengine.ParseStatusTolerant` when found, so the closure keeps its signature and `runReader.read` and `runread_test.go`'s stub are untouched.
  It decodes the same bytes with the same parser, so D5 holds for the caller.

Tests:
- `TestOriginRunReads`'s status subtest asserts the returned bytes equal the status file the subtest committed and pushed, and that an unpushed slug is not found.
  It drops its `shedengine` import and its assertion on the decoded state and on the tolerated unknown key, which is the codec's rule, held by card 2's `TestStatusCodec` and shed's own tests.
- `TestOriginCodeTip` drops its `Machine` assertions, the codec's verdict, which a fabric test takes as fixture and never asserts (D13); it keeps the tip, time and missing reads.
- `TestOriginRecordRules` (core, tier0) walks each branch of `MarkBeamVerdict`, `ReadOriginForVerdict` and `RemoteOnly`; it adds under 1 second to one tier0 run.
  `TestMarkBeam` and `TestAdd_HubReaderBeforeWiringLeavesNoLym` stay as the tests of the reads and writes.
- `TestInstallRequiredSeams` is a tier1 test in `internal/fabriccli_test`, the installer's adapter test against the real shed module.
  It builds a pair through `internal/hubforge` first, then unsets the paths and codec seams through `fabricengine.SetRunRecordPaths(nil)` and `SetStatusCodec(nil)`, restoring each on return, and does not call `t.Parallel`.
  With the seams unset, `fabricengine.OriginRunStatus` and `fabricengine.OriginCodeTip` each refuse naming the run-records paths seam; with only the codec unset, `OriginCodeTip` refuses naming the codec seam.
  It then calls `fabriccli.InstallRequiredSeams` and checks each adapter against the real shed module:
  the paths, by writing the pair's run status through `state.UpdateJSON` over shed's `Status` at `shedrun.StatusFile` in the pair's records worktree, committing and pushing it, and asserting `OriginRunStatus` returns those bytes, found;
  the codec, by planting a transition itself, since the `fabricengine_test` planting helper is out of its package's reach:
  it writes a status holding two history rows through `state.UpdateJSON` over `shedengine.Status` in the pair's records worktree at the status half of `shedrun.TreeRunFileRels(slug)`, the tree path `OriginCodeTip` reads,
  commits it with a message carrying `fabriclayout.TransitionTrailerKey` and `fabriclayout.WarpSHATrailerKey` naming the pair's code tip, pushes and fetches,
  and asserts `OriginCodeTip`'s machine is the second row's.
  The run-lock adapter's check is card 11's, once `tryPrimeLock` reads the seam.
  So each adapter names what shed writes, and the refusal is reached through a setter.

**ImpactSummary:** One re-signed exported reader with one production caller (pairteardown), one unexported reader and one test; originreads.go drops both shed imports.
