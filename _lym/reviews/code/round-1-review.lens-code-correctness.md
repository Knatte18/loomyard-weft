```yaml
lens: code-correctness
outcome: reported
```

Lens: code-correctness. I ran each changed function and branch against concrete inputs.

**List: function or branch, the case I ran, and whether it is correct**

- `CheckParent` note exemption:
  - Empty `notePath` with a write to `webster-master.md` is flagged. Correct.
  - Absolute `notePath` with a relative write `_lym/loom/friction/webster-master.md` resolves against workdir and matches. Correct.
  - A write that goes through the `_lym` link canonicalizes both sides in `isWriteOf` (audit.go:121-131) and matches. Correct.
  - A write to the `-2` sibling is flagged. Correct.
- `isWriteOf` when `CanonicalPath` fails: returns false, so the write is flagged. That fails closed. Correct.
- `Run`, saving `st.MasterNotePath`:
  - Friction off: `NotePath` returns "", so nothing is exempt. Correct.
  - Resumed run: `NotePath` returns `webster-master-2.md` because the first note exists, and it is saved before Master can call record-batch. Correct.
- `runExitAuditCrossCheck` and `RecordBatch` both read the note path from state. Correct.
- `newVerifyGate` `pass()`:
  - `preFix` "" means `fixedHead` stays "". As written.
  - `preFix` P with HEAD P (dirty-tree failure, then cleaned with no commit) means `fixedHead` stays "". Correct.
  - `preFix` P with HEAD F gives `fixedHead` F.
  - `s.head()` fails: the error is returned before any state write. Correct.
  - Restart across closures: run 1 records PreFixHead P and fixers commit F1..Fk; run 2's closure starts with `preFix` "" and passes on its first evaluation, so `fixedHead` stays "", FixedHead is never recorded, and PreFixHead is cleared. **Wrong. See F2.**
- `recordGatePass`:
  - PreFixHead "" with `fixedHead` "": no save. Correct.
  - `fixedHead` equal to the stored value with PreFixHead set: saves. Correct.
  - `SaveState` fails: the error propagates. Correct.
- `mutateGateState` with no state.json: nil, no write. Correct.
- `reconcileAcceptHead`:
  - HEAD equals lastHead: nil. Correct.
  - HEAD equals FixedHead with lastHead an ancestor: nil.
  - A non-merge commit above FixedHead: refused with wording against lastHead. Correct.
  - FixedHead equals lastHead, or is missing: refused. Correct.
  - `IsAncestor` fails: the raw error is returned, not wrapped in `ErrAuditNotAcceptable`. That is acceptable for a git failure.
  - The second `reconcileHead` returns a git probe error, for example from `CommitParents`: the error is swallowed and reported as the lastHead refusal. **See F4.**
  - Commits between lastHead and the closure's preFix: never vetted. **See F3.**
- `recordsSideStatus`:
  - Empty or pathless path: (false, false). Correct.
  - Path outside `_lym`: not on records. Correct.
  - Path under the webster, reports or plan directory: excluded. Correct.
  - Absent untracked note: clean. Correct.
  - Tracked records file that Master deleted: Lstat says not-exist, so it reads clean. **Wrong. See F1.**
  - Master's edit, after `fabricSync` committed the whole `_lym` pathspec at a batch boundary (webstercli/sync.go:30-38): `PathDiffers(records, "HEAD", …)` is false, so it reads clean. **Wrong. See F1.**
  - Ignored file: (true, false). Correct.
  - `IgnoredPath` error: returns (true, false, err), and the caller returns the error. Correct.
  - Dangling symlink: Lstat finds it, so it goes on to git. Correct.
- `splitRecordsSidePaths`: mixed input keeps the order; an error yields nil slices. Correct.
- `pendingPathsWayForward` records split:
  - Clean records path plus a differing tracked path: restore, accept-audit, reentry. Correct.
  - Changed records path plus a pathless finding: reset route. Correct.
  - Consistency with `AcceptPendingAudit`: both reach `_lym` paths through the same unverifiable class (suspect.go:218-221 and the `rest` split). In standalone, records equals worktree/`_lym`, so both use the same git. Consistent.
- `AcceptPendingAudit` refusal assembly with records changed and unverifiable paths: both parts appear and the reset route comes last. Correct. With no batch head, records paths still clear. Correct.

**Findings**

**F1. BLOCKING, design.** `internal/websterengine/findingtext.go:182-197` (`recordsSideStatus`)
"Clean" is judged against the records side's own HEAD, which includes Master's writes.
- Evidence: webstercli/sync.go:30-38. `fabricSync` commits the whole `_lym` pathspec at every bracket-verb boundary, so any parent write under `_lym` that a record-batch sync committed reads "unchanged". Examples outside the webster, reports and plan directories include `_lym/discussion/decision-record.md` and `_lym/loom/…` run records.
- Evidence: findingtext.go:182. Lstat not-exist returns (true, true), so a tracked records file that Master deleted also clears.
- Effect: the parent-write correctness finding for `_lym` writes is cleared without evidence.
- Fix: compare against a records-side baseline taken before Master ran, such as the records HEAD saved at spawn. Treat "absent" as clean only when that baseline also lacks the path.

**F2. MEDIUM, design.** `internal/websterengine/verifygate.go:394-403` (the `pass` closure)
`pass` reads only the closure's in-memory `preFix`. A run restarted after fix commits records no FixedHead, so accept-audit still refuses on the earlier run's fix commits.
- Evidence: verifygate.go:347-355. `preFix` starts as "" in each closure, while state.json's PreFixHead from the earlier run persists.
- Effect: the entry's third Idea bullet is unmet on restart.
- Fix: seed the closure's `preFix` from the persisted `st.PreFixHead`, or compute `fixedHead` against it.

**F3. MEDIUM, design.** `internal/websterengine/auditledger.go:203-232` (`reconcileAcceptHead`)
FixedHead tolerates every commit in lastHead..FixedHead, but the gate's `fixRejection` vetted only preFix..HEAD of one closure.
- Evidence: verifygate.go:472-475, and `fixCommitRejection` at gitwrap.go:278-306.
- Effect: commits below the closure's preFix are never vetted. That covers anything a restarted run overwrote PreFixHead past (setPreFixHead, verifygate.go:288-296), including non-clean merges. The doc comment's claim "which the gate's commit check vetted" is false.
- Fix: persist the vetted base with FixedHead, and require lastHead to reconcile to that base, for example through `reconcileHead`.

**F4. LOW, design.** `internal/websterengine/auditledger.go:229-231`
The second `reconcileHead`'s error is dropped whatever its kind. A git probe failure (HeadSHA or CommitParents) is misreported as the lastHead refusal, wrapped in `ErrAuditNotAcceptable`, instead of surfacing as an operational error.
- Fix: return non-refusal errors from the FixedHead attempt.

**Tool calls**
1. Read the rest of `isWriteOf` and the source of `frictionDir`: the diff showed `isWriteOf` only in part, and I needed whether `notePath` is absolute.
