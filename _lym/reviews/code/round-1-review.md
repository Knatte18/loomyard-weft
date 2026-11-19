---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/runfsm/act.go:147
    summary: "Removing PushOwedWayForward makes the loom act refusal for an owed records push name `lym fabric push` in the records sibling, where the verb refuses; at the base it named the paired code anchor."
    fix: "Restore the general owed-push way forward that the diff removed. Put back runfsm.Inputs.PushOwedWayForward and Verdict.PushOwedWayForward, the copy in Act, the clearing in Judge's PushOwed aside, and the Refusal branch that names it when set. Put back pairteardown runread.go's `inputs.PushOwedWayForward = fabricengine.FailedPushWayForward(owed[0])`, and restore the TestReadRun wantIn and a TestAct row for a set way forward. Use a records-side record with CodeWorktree as the example in place of the removed `git push` in the prime row. Keep operator_unpushed and protected removed. In pattern/PATTERN-run-fsm.md, under the owed-push paragraph, say the act rule's refusal names the owed record's own way forward, as FailedPushWayForward gives it. Leave the PATTERN.md entry line as declared. In _lym/landing/summary.md, the FailedPushWayForward bullet then holds as written."
    origin: assist:assist
  - id: F2
    severity: BLOCKING
    class: consistency
    location: internal/fabriccli/cli_test.go:407
    summary: "TestCommand_CloneEngineRefusalReachesEnvelopeUnchanged ran with t.Parallel at the base but no longer calls it. It touches no env, cwd or shared fixture, against PATTERN-test-speed's rule that a test calls t.Parallel unless it touches process-global state."
    fix: "Add `t.Parallel()` on the line after `tiergate.Require(t, 1)` in TestCommand_CloneEngineRefusalReachesEnvelopeUnchanged in internal/fabriccli/cli_test.go."
    origin: assist:assist
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/fabricengine/boardweft.go:39
    summary: "ensureBoardWorktree loses its adoption of origin/<warpBranch>. A two-repository clone whose weft remote HEAD differs from the warp's default branch, a case clone.go's own comment names, now orphan-creates the Bolt checkout and drops the board's history."
    fix: "Restore in ensureBoardWorktree the base's remote-branch path. Read refs/remotes/origin/<warpBranch> with RefSHA, treat ErrRefNotFound as absent, and when the branch is absent locally but present on origin, run `worktree add --track -b <warpBranch> <boardPath> origin/<warpBranch>` through containedWorktreeAdd. Orphan-create only when both refs are absent, and restore the doc sentence. Add a row to the existing CloneHub adoption test whose weft remote HEAD is not the warp's branch, asserting the Bolt checkout holds origin's board commit."
    origin: assist:assist
  - id: F4
    severity: MEDIUM
    class: scope
    location: internal/loomcli/wiring_commitstatus_integration_test.go#TestCommitStatusSeam_Real
    summary: "The diff deletes c83fec49's two-repository subtest `records cherry-pick in progress commits the transition` with its one-repository neighbours. This is the only seam-level pin of D7's acceptance criterion, and no decision licenses deleting two-repository coverage; no Loosened-Check trailer names it."
    fix: "Restore in internal/loomcli/wiring_commitstatus_integration_test.go the subtest `records cherry-pick in progress commits the transition` as it stood at 7346d6a8. Restore its helpers startStoppedCherryPickInRecords and runGitAllowingFailure, and the os/exec import they need. Keep the one-repository wait rows deleted."
    origin: lens:code-tests
  - id: F5
    severity: LOW
    class: consistency
    location: internal/battencli/overview_test.go:716
    summary: "The warn check widens from the thresholds warning's own text to any WARN. The wantWarn row now passes on any warning, with no Loosened-Check trailer naming it."
    fix: "In TestLoadOverviewThresholds, assert a warning by its own text when tt.wantWarn is true: `loading the orch thresholds for the overview failed`. Keep the check that the log holds no WARN at all when tt.wantWarn is false."
    origin: handler
  - id: F6
    severity: LOW
    class: scope
    location: internal/fabricengine/doc.go#MergeStateActive
    summary: "The diff drops c83fec49's still-true two-repository rationale: a weft cherry-pick, revert or rebase does not count, since fabric's own replay cherry-picks under the write lock the commit waits on. D7 keeps that rule, and the text is no records-on-code-branch text."
    fix: "In internal/fabricengine/doc.go, after the sentence that introduces `MergeStateActive` as the weft-only mid-merge probe, add two sentences. First, that a weft cherry-pick, revert or rebase does not count. Second, that fabric's own replay cherry-picks under the write lock the commit waits on."
    origin: lens:code-scope
  - id: F7
    severity: LOW
    class: consistency
    location: _lym/landing/summary.md:56
    summary: "The Merge danger blast radius names only fabric. The diff also changes lymcwd's resolvers, the statuscommit core every transition commit runs through, landingshed's Finalize and the runfsm act refusal."
    fix: "In _lym/landing/summary.md's Merge danger section, widen Blast radius to name fabric, lymcwd resolution, the statuscommit transition commit (loom and batten), landingshed Finalize and the runfsm act refusal."
    origin: handler
---

The range is `7346d6a8..HEAD` (4dcf7cd7 and 91b285da).
Against `f06acf39^`, HEAD's files differ only by the later commits' unrelated edits, the D6 keeps and the new D7 test rows.
So the removal is a faithful revert of f06acf39, and the declared PATTERN entries land verbatim.
The findings are where that revert also undid general two-repository behaviour, or dropped coverage that f06acf39 or c83fec49 had added.

### [BLOCKING] Owed records push names a refused verb directory

**Location:** internal/runfsm/act.go:147

**Issue:** For a records-side failure, `pairteardown.runReader.read` fills `PushOwedWorktree` from `owed[0].RepoPath`, which `pushanchored.go:308` sets to the weft path.
`Verdict.Refusal` now renders "run `lym fabric push` in <records sibling>".
`RequireWarpWorktree` (`internal/fabricengine/fabric.go:121`) refuses every fabric verb there.
At the base, `FailedPushWayForward` resolved the step through `pushStep`, which names `lymVerbDir(f.CodeWorktree)` for the records side (`internal/fabricengine/failedpush.go:346`).
That propagation served every hub, not only one-repository ones.
D5 removes only what f06acf39 added "only to keep records on a code branch", and the change description says `FailedPushWayForward` "stays with all its callers".
The loom refusal now misdirects the operator, against PATTERN-refusal-way-forward.

### [BLOCKING] Clone-refusal test lost t.Parallel

**Location:** internal/fabriccli/cli_test.go:407

**Issue:** At the base the test called `t.Parallel()` at top level and in each row, so it is safe to run in parallel.
The flattened version calls neither and states no process-global state.
PATTERN-test-speed's background, `pattern/PATTERN-test-speed.md:190`, requires `t.Parallel` unless a test touches env, cwd or a shared fixture.

### [MEDIUM] Bolt checkout no longer adopts a remote-only board branch

**Location:** internal/fabricengine/boardweft.go:39

**Issue:** `CloneHub` reads `warpBranch` from the warp clone and states that "The weft remote's HEAD may be an empty bare created with a different default branch" (`internal/fabricengine/clone.go:289-295`).
When that HEAD differs, the weft clone holds `<warpBranch>` only as `origin/<warpBranch>`.
The diff removes the `--track` adoption, so `ensureBoardWorktree` orphan-creates the board, and `CloneAndWire` reconciles config against an empty Bolt.
The adoption is not a one-repository branch, so D5 does not license removing it.

### [MEDIUM] Two-repository cherry-pick seam test deleted

**Location:** internal/loomcli/wiring_commitstatus_integration_test.go#TestCommitStatusSeam_Real

**Issue:** c83fec49 added this subtest for a two-repository hub.
It runs the real probe during a stopped records cherry-pick and asserts that the transition commits.
`TestMergeStateActive` pins only the probe's answer, not the seam's commit.
The acceptance criterion "D7: a status commit during a cherry-pick in the records worktree commits instead of skipping" loses its only end-to-end pin.
The code-tests and code-scope lenses and the assistant each raised this finding.

### [LOW] Overview thresholds warn check widened

**Location:** internal/battencli/overview_test.go:716

**Issue:** The base asserted the thresholds warning by its own text.
The diff reverts to `strings.Contains(logs.String(), "WARN")`, so the invalid-config row passes on any warning.
That is stricter for the no-warn rows but looser for the warn row.

### [LOW] MergeStateActive rationale dropped from the fabric doc

**Location:** internal/fabricengine/doc.go#MergeStateActive

**Issue:** The removed sentences state D7's kept rule and why it holds, and they mention no one-repository layout.
G16 removes only records-on-code-branch text.

### [LOW] Merge danger blast radius too narrow

**Location:** _lym/landing/summary.md:56

**Issue:** "Blast radius: fabric" omits these packages the diff changes:
- lymcwd's `Resolve`/`ResolveWithin`;
- statuscommit, which every loom and batten transition commit runs through;
- landingshed Finalize;
- the runfsm refusal.

The `two-way` door call itself holds.
Only one-repository hubs, which the record assumes none of, wrote the removed classes and the `lym.primeBranch` key.

## Rejected

- code-tests F2 (batten `TwoRepositoryMergeInProgressSkips` row): the batten seam passes `MergeActive` straight to the statuscommit core, whose skip `TestNew_MergeActiveSkips` pins. The deleted test existed only to pin the layout dispatch the diff removes, and no such test existed before f06acf39.
- code-affected-set F1 (the summary's "stays with all its callers" claim): merged into F1, whose fix restores the pairteardown caller and so makes the claim true.
- assist F4 (clone hook-install mutation survivor at clone.go:275): the line is byte-identical to f06acf39^, so the diff adds no behaviour there. The survivor shows an older coverage gap, not a gap in this diff.
- Mutation survivors at commit.go:112/115: the refusal order is pinned by `mergesiblings_integration_test.go:103` and `:290`, which the mutation tier did not run. The survivors at pushanchored.go:192, cleanupremotewarp.go:119, add.go:278, commitweftpaths.go:66, forkrecords.go:45 and spawn.go:163 fall on lines restored verbatim from f06acf39^, so the diff adds no behaviour there.
