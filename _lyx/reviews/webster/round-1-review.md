---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: internal/websterengine/audit.go:356
    summary: ClassifyViolation classes every write in this worktree as correctness, git-ignored ones included, when this worktree lies inside another worktree of the repository
  - id: F2
    severity: MEDIUM
    location: internal/websterengine/batchfail.go:63
    summary: a recovery re-failed by the suspect check re-probes each Blob from the current worktree, so the audit-flagged blob is overwritten and an uncommitted delete resets it to empty, disabling the flagged-content check for the next recovery
  - id: F3
    severity: MEDIUM
    location: internal/websterengine/runlevel.go:725
    summary: the --fresh drop warnings reach only the done path, so a fresh run that drops findings and then ends asking, died, timed out or errored drops them silently
  - id: F4
    severity: LOW
    location: internal/webstercli/rebaseline.go:49
    summary: the --card usage error carries no way-forward clause, while the refusal-spec row claims one
  - id: F5
    severity: LOW
    location: internal/websterengine/rebaseline.go:47
    summary: doc comments and help text still describe the pre-card behaviour of the fingerprint restamp, the validate re-baseline persist, the suspect-path file and run --fresh
  - id: F6
    severity: LOW
    location: internal/websterengine/doc.go:113
    summary: new and changed Go comments from cards 47-53 and e007a2bac break mid-sentence or at fixed columns, against CONSTRAINTS.md's Comment Line-Break Convention
  - id: F7
    severity: NIT
    location: cmd/lyx/rawgitmutation_test.go:46
    summary: the allowlist reason attributes plain status --porcelain to ignoredPath, which runs status --porcelain --ignored
  - id: F8
    severity: NIT
    location: internal/websterengine/fingerprint.go:28
    summary: fingerprint re-implements planFileNames' file-set selection, so the same-file-set guarantee planFileHashes relies on is held by two copies
  - id: F9
    severity: NIT
    location: internal/websterengine/auditledger.go:96
    summary: AcceptPendingAudit does not dedupe suspect paths across findings, so a path two findings name is listed twice in the refusal
---

Review range: `_lyx/shed/refusal-audit/status.json` gives `product.parent = main`;
the range is `git diff 1c5618424..HEAD` (95 commits, cards 1–54 plus fix commits).
`go build ./...`, `go vet -tags smoke ./...`, `go test ./...` and `go test -tags integration` over `internal/websterengine`, `internal/webstercli` and `cmd/lyx` all pass.

### [MEDIUM] A nested worktree makes every write in this worktree correctness

**Location:** `internal/websterengine/audit.go:352-364` (`ClassifyViolation`), with `otherWorktrees` in `internal/websterengine/gitwrap.go:203`.

**Issue:** The other-worktree loop runs before the own-worktree and git-ignored checks, and tests `pathWithin(other, written)` for every listed worktree other than this one.
When this run's worktree lies inside another worktree, for example a Claude Code worktree under `<main>/.claude/worktrees/<name>` or any task worktree created beneath the main checkout, the enclosing worktree's root contains every path in this worktree.
Every Master parent-write then returns correctness at the loop, including a git-ignored build output that should fall through to the policy rule at line 372.
That fails the batch, or holds run exit, on a false positive, the kind of wedge this branch exists to remove.
The rule should go to the innermost worktree that holds the write, so that an enclosing worktree does not claim a path that belongs to this one.
Porcelain entries for bare, prunable and missing worktrees are handled: each is a `worktree <path>` line, and `canonicalPath` resolves a missing path through its nearest existing ancestor instead of failing.

**Fix:** In the loop, skip an `other` that contains this worktree when the write also lies within this worktree, so the innermost worktree decides.
A worktree nested inside this one still classes as another worktree.
Add a `TestClassifyViolation` row with a worktree nested inside the main checkout, where a git-ignored write in it stays policy.

### [MEDIUM] Re-failing a recovery overwrites the flagged blob

**Location:** `internal/websterengine/batchfail.go:63-90` (`failBatch`), reached from `internal/websterengine/recoverbatch.go:374`.

**Issue:** `failBatch` always recomputes `suspectBlobs` from the worktree as it is now, and replaces `Batch.SuspectPaths` with the result.
On the first failure that is the flagged content.
On a re-failure from `PersistRecoveryTerminal`'s suspect check, it is whatever the recovery strand left behind.
The card says the batch fails again "with the same suspect paths, so the next `recover-batch` checks them again", but the blob being checked has changed.
For example, the flagged `internal/x.go` was committed at the failed batch's head.
The first recovery deletes it in the worktree without committing; the head-diff check fails the batch, and the re-probe records `Blob: ""` because the file is absent.
The second recovery restores the file with `git checkout` and commits other work.
Now the head-diff check passes, and the flagged-content check is skipped because `blobs[p] == ""`, so the flagged content is recorded done.

**Fix:** In `failBatch`, keep the recorded blob for every path the batch already records, so the blob the audit first flagged stays the one later recoveries check.
Pin the case with a test where a recovery deletes the path uncommitted, then a second recovery restores and commits the flagged content.

### [MEDIUM] The --fresh drop is silent on every non-done outcome

**Location:** `internal/websterengine/runlevel.go:454` (`freshPendingDrop`), `runlevel.go:725`.

**Issue:** `freshWarnings` is prepended to `RunResult.Warnings` only inside the `OutcomeDone` branch.
After the drop, `Run` archives `state.json`, writes a fresh one and goes on to the validation refusal, the spawn, and Master's wait.
Every later return path, such as a plan-validation refusal, a spawn failure, `MasterAskingError`, `MasterDiedError`, `MasterTimeoutError` or an unknown outcome, returns the zero `RunResult` and drops the warnings.
The new `state.json` no longer holds the findings, so a re-run cannot report them either.
That contradicts card 52's "the drop is never silent".
The refusal itself fires before any archive (`freshPendingDrop` runs before the `switch`), and `--fresh` on an unchanged plan with no pending findings stays a no-op (`freshDrop` false, so the `switch` matches nothing).

**Fix:** Log each warning through `logger.Warn` when the drop is committed, as the integration stage already does for warnings no envelope can carry, and keep the `RunResult` prepend for the done path.

### [LOW] The --card usage error has no way forward

**Location:** `internal/webstercli/rebaseline.go:49`, and the `rebaseline bad card number` row in `contracts/specs/refusal-spec.md`.

**Issue:** The message is `webster: --card "x" is not a card number; pass a positive integer such as 5 or 05`.
It has no trailing `way forward:` clause, which the Shared Decision way-forward-phrasing requires.
The refusal-spec cell (`re-run lyx webster rebaseline naming card numbers`) quotes a clause the code does not carry, while the file's own rule says each cell quotes the message's clause.

**Fix:** End the message with `; way forward: re-run "lyx webster rebaseline" naming each changed card with --card NN, such as --card 5 or --card 05`.
Make the cell quote that clause, and assert `way forward` in `TestRebaselineCmd_RefusesNonNumericCard`.

### [LOW] Stale doc comments and help text after cards 50–52

**Location:** `internal/websterengine/rebaseline.go:47`, `internal/websterengine/fingerprint.go:127`, `internal/webstercli/cli.go:298-299`, `internal/websterengine/suspect.go:1-3`, `internal/webstercli/run.go:59-61`.

**Issue:**
- `Rebaseline`'s doc says that otherwise it restamps `State.PlanFingerprint` and leaves every other field untouched; `restampFingerprint` now also rewrites `State.PlanFileHashes`.
- `restampFingerprint`'s first line names only `st.PlanFingerprint`.
- `persistPlanFingerprintRebaseline` says it persists the fingerprint "and NOTHING ELSE" and that the state it writes "carries only the new fingerprint"; it now copies `PlanFileHashes` too.
- `suspect.go`'s header describes a run-exit-only check against "the last batch head".
  The file now also holds the recovery check (`suspectBlobs`, `checkRecoveredSuspects`) and checks against a caller-chosen base: the run's start commit for `--fresh`, and the report head for recovery.
- `run`'s `Long` still says `--fresh` "starts over ONLY on that mismatch" and is a no-op on an unchanged plan, then contradicts itself in the appended clause.

**Fix:** Update each comment and the help text to state the current behaviour.

### [LOW] Comment line-break convention violations

**Location:** `internal/websterengine/doc.go:113-120`, `doc.go:162-163`, `internal/websterengine/state.go:128-130`, `state.go:184-185`, `internal/websterengine/gitwrap.go:7-9`, `gitwrap.go:203-204`, `internal/websterengine/audit.go:80-81`, `audit.go:138-139`, `audit.go:312-315`, `internal/websterengine/runlevel.go:1063-1064`, `internal/websterengine/recordbatch_test.go:648-649`, `recordbatch_test.go:916-917`.

**Issue:** CONSTRAINTS.md's Comment Line-Break Convention wants one sentence per line, with breaks only at independent-clause boundaries, for every new or changed comment.
- `doc.go:113-120` (card 53's recover-batch paragraph) and `state.go:128-130` (`PlanFileHashes`) are new fixed-column wraps.
- `gitwrap.go:7-9` and `audit.go:138-139` are changed lines that keep a legacy mid-phrase break ("gitexec.Run / directly", "contract files / (outcomePath or summaryPath)").
- The rest break at a relative clause, a participle, an appositive, a compound predicate or a list alternative, none of which is an independent clause: ", which only webster…", ", read from…", ", falling back…", ", empty when…", ", a read-only one…", ", archives its report…", ", or under webster's scratch…", ", or a fabric reference…".

**Fix:** Rewrite each block as one sentence per line, splitting long sentences into several sentences rather than wrapping them.

### [NIT] Allowlist reason names the wrong probe for ignoredPath

**Location:** `cmd/lyx/rawgitmutation_test.go:46`.

**Issue:** The reason lists `status --porcelain` for both `dirty` and `ignoredPath`, but `ignoredPath` runs `status --porcelain --ignored`.
Every other probe named matches the command `gitwrap.go` runs.
The doc comments name the right file: `gitwrap.go`'s header lists the moved probes, and no comment still places them in `suspect.go`.

**Fix:** List `status --porcelain` (dirty) and `status --porcelain --ignored` (ignoredPath) separately.

### [NIT] fingerprint duplicates planFileNames

**Location:** `internal/websterengine/fingerprint.go:28-41`.

**Issue:** `planFileHashes` relies on hashing exactly the set `fingerprint` reads, but the selection (every `.md` file but the amendments file) is written twice.
The two match today, but nothing keeps them matching.

**Fix:** Have `fingerprint` call `planFileNames`.

### [NIT] Duplicate paths in the accept-audit refusal

**Location:** `internal/websterengine/auditledger.go:96-103`.

**Issue:** Paths from every pending finding are concatenated without deduplication, so two findings that name the same file list it twice in the differing or not-checkable list.
`freshPendingDrop` already dedupes.

**Fix:** Skip a path already collected.

## Focus verdicts that are not defects

- Card 51, `lastBatchHead` and `runStartCommit`:
  pending findings are recorded only by `runExitAuditCrossCheck`, which `mapMasterDone` reaches only after `verifyEveryBatchDone` passes, and run entry is blocked while they are pending.
  When `accept-audit` runs, every batch is therefore terminal done, so a failed batch's `HeadSHA` cannot become the base.
  `recoverSpawn` carries `prior.StartSHA`, so the lowest-numbered `StartSHA` stays the run's start after a failure and recovery.
- Card 53, `checkRecoveredSuspects`:
  - an uncommitted suspect path is caught by the head diff;
  - committed flagged content is caught by the blob comparison;
  - a reverted or re-derived path passes;
  - a path with an empty `Blob`, or one outside the tracked tree, skips the blob check;
  - `commitBlob` is called with the worktree-relative `rel` from `trackedRel`.
  `StartSHA` is never empty on a recovery record.
  The defect is the re-fail blob overwrite (F2).
- Card 50: `planFileHashes` reads the fingerprint's file set.
  Both `State` builds in `Run`, `restampFingerprint` and `persistPlanFingerprintRebaseline` set `PlanFileHashes`.
  `Rebaseline`'s NN parse, the `00-overview.md` refusal and the unnamed-card refusal match the card's messages, apart from F4.
- Commit e007a2bac: every probe now goes through `gitexec.Run` in `gitwrap.go`, and the doc comments name the right file (see F7).
- Card 54: every edited row quotes the clause its message carries, apart from F4, and each refusal is reached by a test (the `RefusesNonNumericCard`, `RefusesUnnamedCard`, `AcceptAuditCmd_RefusesDifferingPath`, `FreshRefusesWhileSuspectPathDiffers` and `PersistRecoveryTerminal_*` tests).
- Per-card mechanical checks:
  - card 47's deleted functions and every table declared beside them (`readOnlyGit`, `readOnlyFabric`, `readOnlyTools`, `loopHeaders`, `gitValueOptions`, `shellKeywords`, `commandWrappers`, the value-option sets, `systemBinDirs`, `harmlessRedirectTargets`) and its deleted tests have no remaining references in `internal` or `cmd`, and neither does card 51's `dispositionAccepted`;
  - card 51's Create targets `suspect.go` and `suspect_test.go` exist, the test file tagged `integration`;
  - every test the cards 47–53 name exists, and both suites pass.
