---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-correctness
    location: internal/websterengine/findingtext.go#recordsSideStatus
    summary: "A records-side path counts as clean when it matches the records worktree's own HEAD, which already holds Master's write once fabricSync commits the whole _lym pathspec at a batch boundary; a deleted tracked records file also reads as clean because it is absent. Any parent write under _lym outside webster, reports and plan, such as the decision record or loom run records, is then accepted without evidence."
    fix: "Record a records-side baseline before Master can write. At spawn in Run (runlevel.go, beside st.MasterNotePath), save the records worktree's HEAD SHA as a new State field, for example MasterRecordsBase, using a read-only rev-parse through the Git seam. In recordsSideStatus, compare the path against that baseline commit instead of HEAD: the path is clean when the baseline does not hold it and it is absent now, or when its current content matches the baseline's blob (worktreeBlob versus commitBlob against the records directory). A path the baseline holds that is now absent is changed. An empty baseline, from a state written before the field existed, leaves the path unverifiable so it keeps the reset route. Make recordsRestoreStep name the baseline commit, as restoreStep names the batch head. Clear the field wherever a fresh State is built. Add rows to TestAcceptPendingAudit_RecordsSidePath for a note committed to the records repository after the baseline (refused) and for a tracked records file deleted after it (refused)."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/websterengine/verifygate.go#newVerifyGate
    summary: "pass() derives the fixed head only from the closure's in-memory preFix. After a restart between the failed and the passing evaluation, the new closure starts with preFix empty, so the pass records no FixedHead and clears the persisted PreFixHead. accept-audit then still refuses on the earlier run's fix commits, so the entry's third Idea bullet is not met for a restarted run."
    fix: "In NewVerifyGate, read state.json's PreFixHead when the gate is built and pass it to newVerifyGate as the closure's initial preFix, through a new seam or parameter, so a restarted gate keeps the earlier failure's base. The fixRejection check then covers the earlier fix commits too, and pass() records HEAD as FixedHead when HEAD moved above it. Add a TestVerifyGate row that builds the gate with a persisted pre-fix head, passes at a moved HEAD on its first evaluation, and asserts that recordPass received that HEAD."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/websterengine/auditledger.go#reconcileAcceptHead
    summary: "reconcileAcceptHead tolerates every commit between the last batch head and FixedHead, but the gate's commit check (fixCommitRejection) vetted only the commits above that closure's pre-fix head. A commit between the last batch head and the pre-fix head, such as a non-clean merge that landed before the gate first failed, is accepted unchecked. The doc comments' claim that the gate vetted those commits is false."
    fix: "Persist the vetted base next to FixedHead: have recordGatePass also store the closure's preFix as a new State field, for example FixedBase, cleared wherever FixedHead is cleared (webstercli/reset.go and a fresh State). In reconcileAcceptHead, accept the FixedHead route only when FixedBase is set and reconcileHead(git, root, lastHead, ...) succeeds with FixedBase standing in for HEAD, i.e. only clean parent merges lie between lastHead and FixedBase; walk that segment with the same first-parent merge check reconcileHead uses. Then reword the reconcileAcceptHead, AcceptPendingAudit, State.FixedHead and doc.go comments to say which range the gate vetted. Add a TestAcceptPendingAudit_AcceptsVerifyFixCommits row where a non-merge commit sits between the last batch head and FixedBase and expect ErrAuditNotAcceptable."
  - id: F4
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: contracts/stencils/webster/webster-template-master.md:321
    summary: "The changed rule at line 319 lets Master write its friction note, but line 321 in the same section still says 'you write only your two contract files' and line 308 says 'never write outside your two contract files', so the stencil contradicts the write the entry's first Idea bullet allows."
    fix: "In webster-template-master.md, change line 321's closing clause to 'and you write only your two contract files and the friction note, when one is named', and change line 308's 'never write outside your two contract files' to 'never write outside your two contract files and the friction note'."
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/websterengine/runlevel.go:805
    summary: "No test pins that Run saves State.MasterNotePath at spawn or that record-batch and the run-exit audit pass it to CheckParent; the mutation report's survivor at runlevel.go:805 shows that dropping the save, which brings the reported bug back, passes every test."
    fix: "In an existing TestRun_* that spawns Master (for example TestRun_MasterSpawn), assert the saved state.json carries MasterNotePath equal to the note path in the rendered prompt. Add a row to TestRun_DoneOutcome (or TestRecordBatch_AuditOutcomes) whose parent transcript writes that saved path and assert no parent-write finding results."
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/websterengine/auditledger.go#reconcileAcceptHead
    summary: "The FixedHead attempt discards every error from its reconcileHead call, so a git probe failure such as HeadSHA or CommitParents is reported as the last-batch-head refusal wrapped in ErrAuditNotAcceptable, sending the operator to a reset instead of surfacing the operational error."
    fix: "In reconcileAcceptHead, keep the reconcileHead error of the FixedHead attempt; return nil when it is nil, fall through to the lastHead refusal only when it is a head refusal (have reconcileHead mark refusals with a sentinel error, or compare against its refusal construction), and return any other error as is."
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/websterengine/suspect_test.go#TestAcceptPendingAudit_AcceptsVerifyFixCommits
    summary: "reconcileAcceptHead's refusing branches have no row although the focus asks for them: a FixedHead the last batch head does not precede, one equal to it, and one missing from the repository."
    fix: "Add rows to TestAcceptPendingAudit_AcceptsVerifyFixCommits, converted to a table, for a FixedHead on a side branch that HEAD reaches only through a merge, a FixedHead equal to the last batch head with a non-merge commit above, and a FixedHead that names a SHA absent from the repository; each expects ErrAuditNotAcceptable naming reset --to last-batch-head and leaves PendingAuditFindings unchanged."
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/websterengine/verifygate_test.go#TestGateStateWrites
    summary: "Two gate branches are unpinned: pass() recording no fixed head when HEAD did not move above preFix, and recordGatePass saving when only FixedHead changes (the mutation survivor at verifygate.go:305, since every step that passes a fixed head starts with PreFixHead set)."
    fix: "Add a TestGateStateWrites step that calls recordGatePass with a new fixed head while PreFixHead is already empty and expects the new FixedHead. In TestVerifyGate_DirtyTreeRecordsPreFixHead, or a sibling gate test, pass after a failure with HEAD unchanged and assert f.fixedHeads equals a one-element slice holding the empty string."
  - id: F9
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/webstercli/reset_test.go:393
    summary: "Nothing tests that lym webster reset clears FixedHead; the reset test asserts only PreFixHead, though the change description claims both are cleared."
    fix: "In the reset test that asserts PreFixHead is cleared (reset_test.go near line 393), seed st.FixedHead beside PreFixHead and assert both are empty after the reset."
  - id: F10
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/websterengine/audit_test.go#TestCheckParent
    summary: "The link-resolved spelling of Master's note, which CheckParent's doc and the change description claim is matched, has no row; every TestCheckParent path is a fake /hub path, so isWriteOf's canonical branch is never reached for the note."
    fix: "Add a TestCheckParent case with a temporary workdir whose _lym is a directory link (fslink.CreateDirLink) to a second temporary directory, notePath given under workdir/_lym, and a parent write naming the note through the link target's real path; expect no violation."
  - id: F11
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: contracts/orchrole/orch-role.md:504
    summary: "The orch role's table still says Webster's Master edits only its two contract files, which the diff changed to include its friction note."
    fix: "In contracts/orchrole/orch-role.md line 504, say Master edits only its two contract files and its friction note."
  - id: F12
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: contracts/stencils/webster/webster-template-master.md:319
    summary: "The new rule names 'the friction note the section above names', but the section directly above is the fabric-sync section; the note path is rendered by the friction directive near the top of the prompt."
    fix: "In line 319, replace 'the friction note the section above names, when it names one' with 'the friction note the Friction note section names, when the prompt has that section'."
  - id: F13
    severity: LOW
    class: scope
    origin: lens:code-scope
    location: internal/webstercli/acceptaudit.go:24
    summary: "The diff rewrites accept-audit help sentences no goal point touches, dropping the fact that a repeated call is harmless; no test or trailer requires the shortening."
    fix: "In acceptAuditCmd's long help, restore the removed sentences ('When it accepts on an absent contract file the envelope also carries next, the step that has Master write both files again.', 'With nothing pending it succeeds and reports an empty accepted list, so a repeated call is harmless.', and the accepted-envelope sentence) and keep only the added verify-gate and records-side clauses."
  - id: F14
    severity: NIT
    class: design
    origin: lens:code-tests
    location: internal/websterengine/suspect_test.go#TestAcceptPendingAudit_RecordsSidePath
    summary: "No row checks that an ignored records-side path counts as changed, or that a path under the reports directory takes the reset route like the webster run state."
    fix: "Add an Ignored row (a .gitignore in the records repository covering the written path, expecting the restore refusal) and a ReportsDir row (a path under the fixture's reports directory, expecting '1) lym webster reset --to start') to TestAcceptPendingAudit_RecordsSidePath."
---

The diff builds all three Idea bullets of the board entry.
One hole blocks it: the records-side check (F1).
Two gaps in the fixed-head tolerance (F2, F3) leave the third bullet unmet on restart and wider than the gate vetted.

### [BLOCKING] Records-side "unchanged" is judged against a HEAD that already holds Master's write

**Location:** `internal/websterengine/findingtext.go#recordsSideStatus`
**Issue:** `recordsSideStatus` runs `PathDiffers(records, "HEAD", canon)`.
`fabricSync` (`internal/webstercli/sync.go`) stages and commits the whole `_lym` pathspec at every batch-boundary commit point.
A parent write under `_lym` made before a record-batch is therefore in the records HEAD when `accept-audit` runs, and reads as unchanged.
The `CommittedNote` row of `TestAcceptPendingAudit_RecordsSidePath` cannot tell a note committed before the run from one Master wrote and a sync committed.
An `os.Lstat` not-exist answers clean too, so deleting a tracked records file clears.
The exclusion of webster, reports and plan directories covers the state file the summary's design decision names.
It does not cover the decision record, the landing summary, review files or loom run records, which other segments read as evidence.
The `parent-write` correctness finding for `_lym` writes is thereby cleared without evidence.

### [MEDIUM] A restarted gate never records the earlier run's fix commits as FixedHead

**Location:** `internal/websterengine/verifygate.go#newVerifyGate`
**Issue:** `preFix` starts empty in every closure while state.json's `PreFixHead` persists.
Suppose run 1's gate fails, fixers commit, and the run stops.
Run 2's gate then passes on its first evaluation with `preFix` empty, so `recordPass("")` clears `PreFixHead` and records no `FixedHead`.
`accept-audit` then refuses on run 1's fix commits exactly as before the change.

### [MEDIUM] FixedHead tolerates commits the gate's commit check never saw

**Location:** `internal/websterengine/auditledger.go#reconcileAcceptHead`
**Issue:** The gate runs `fixRejection(preFix)` only over the commits above the closure's pre-fix head (`verifygate.go`, the `preFix != ""` block; `fixCommitRejection` in `gitwrap.go`).
`reconcileAcceptHead` accepts HEAD whenever the last batch head is an ancestor of `FixedHead`, so every commit from the last batch head up to the pre-fix head goes unchecked.
That includes a non-clean merge, and a span a restarted closure's overwritten `PreFixHead` skips.
The comments on `reconcileAcceptHead`, `State.FixedHead` and doc.go state that the gate vetted that whole range.

### [MEDIUM] The Master stencil still limits Master to its two contract files

**Location:** `contracts/stencils/webster/webster-template-master.md:321`
**Issue:** Line 319 now allows the friction note.
Line 321 ("you write only your two contract files") and line 308 ("never write outside your two contract files") still forbid it in the same prompt.

### [MEDIUM] The saved note path and its two readers have no test

**Location:** `internal/websterengine/runlevel.go:805`
**Issue:** No test reads `MasterNotePath`.
The mutation survivor at `runlevel.go:805` confirms that dropping the save passes every test, though that save is what fixes the reported incident.

### [LOW] A git failure in the FixedHead attempt reads as a refusal

**Location:** `internal/websterengine/auditledger.go#reconcileAcceptHead`
**Issue:** The `if _, err := reconcileHead(...); err == nil` form drops a probe error, and the function returns the last-batch-head refusal in its place.

### [LOW] reconcileAcceptHead's refusing branches are untested

**Location:** `internal/websterengine/suspect_test.go#TestAcceptPendingAudit_AcceptsVerifyFixCommits`
**Issue:** No row covers a `FixedHead` that the last batch head does not precede, one equal to it, or one missing from the repository.

### [LOW] Two verify-gate state branches are unpinned

**Location:** `internal/websterengine/verifygate_test.go#TestGateStateWrites`
**Issue:** No test covers a pass at an unmoved HEAD recording no fixed head, nor a FixedHead-only change saving state (the survivor at `verifygate.go:305`).

### [LOW] Reset's FixedHead clear is untested

**Location:** `internal/webstercli/reset_test.go:393`
**Issue:** The test asserts only `PreFixHead`.

### [LOW] The link-resolved note spelling has no row

**Location:** `internal/websterengine/audit_test.go#TestCheckParent`
**Issue:** Every row uses fake `/hub` paths, so the canonical comparison is never reached for the note.

### [LOW] orch-role.md still names only the two contract files

**Location:** `contracts/orchrole/orch-role.md:504`
**Issue:** It describes Master's write surface, which the diff widened.

### [LOW] "The section above" points at the wrong section

**Location:** `contracts/stencils/webster/webster-template-master.md:319`
**Issue:** The section directly above is the fabric-sync one; the friction directive renders near the top of the prompt.

### [LOW] Unrequested rewording of the accept-audit help

**Location:** `internal/webstercli/acceptaudit.go:24`
**Issue:** The rewording drops the idempotency fact, and neither the entry nor a test asks for it.

### [NIT] Ignored and reports-directory records-side rows are missing

**Location:** `internal/websterengine/suspect_test.go#TestAcceptPendingAudit_RecordsSidePath`
**Issue:** Neither branch has a row.

## Rejected

- Merge danger's two-way door: the diff adds only `omitempty` keys to state.json, which PATTERN-tolerant-records lets an older binary read and keep. An older binary's reset that leaves `FixedHead` set is harmless too, since the ancestry check rejects a `FixedHead` off the new line.
- `AcceptPendingAudit` and `pendingPathsWayForward` classifying paths differently (focus entry 5): `CheckSuspectPaths` and `uncheckableReason` both send every `_lym` path, lexical or canonical, to the unverifiable class. Plan paths are split off first in both. The two therefore classify every path alike, the standalone layout included.
- The mutation survivors at `findingtext.go:162-181`, `auditledger.go:170` and `runlevel.go:1305`: rows of `TestAcceptPendingAudit_RecordsSidePath` kill them, a tier-1 test the coverage map does not hold. The error-return survivors cannot reasonably be killed.
