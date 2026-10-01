---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    location: internal/websterengine/runlevel.go:1049
    summary: a recovery record drops the batch's ForkTranscripts, so the run-exit audit maps the original fork to the integration report and flags its write of its own batch report as fork-state-write
  - id: F2
    severity: MEDIUM
    location: internal/websterengine/recordbatch.go:412
    summary: auditTerminalFork attributes a later batch's or the integration fork's unseen transcript to the terminal batch, failing a done batch and stealing the later batch's transcript
  - id: F3
    severity: MEDIUM
    location: internal/websterengine/runlevel.go:1155
    summary: run --fresh falls back to HEAD when starts are recorded but none qualifies by ancestry, so any HEAD passes the HEAD rule the card limits to a run with no recorded start
  - id: F4
    severity: MEDIUM
    location: internal/websterengine/runlevel.go:1103
    summary: the run-exit and pending-findings way forward names a git restore and accept-audit for paths nothing can check (fork-state-write report paths, .lyx, scratch), and the refusal-spec fork-state-write cell quotes a --fresh clause the code never emits
  - id: F5
    severity: LOW
    location: internal/websterengine/auditledger.go:125
    summary: accept-audit's HEAD-past-head refusal reuses record-batch's text (the report's head_sha, re-run this verb, redo the merge after the batch is recorded), so it never names lyx webster accept-audit as its refusal-spec row says
  - id: F6
    severity: LOW
    location: internal/websterengine/suspect.go:409
    summary: the recovered-blob search is skipped when trackedRel rejects the suspect path at recovery time, and an empty StartSHA makes commitBlob read the index instead of reporting the path absent
  - id: F7
    severity: LOW
    location: internal/websterengine/gitwrap.go:1
    summary: gitwrap.go's header and the raw-git guard's allowlist reason do not name the probes this branch added (ls-tree in treePathsWithBlob, SHAExists and IsAncestor via gitrepo.New)
  - id: F8
    severity: LOW
    location: internal/websterengine/planbaseline.go:48
    summary: storePlanBaseline writes the bytes read now under a hash computed earlier without checking they match, so a plan edited in between leaves a permanently wrong copy under that hash
  - id: F9
    severity: LOW
    location: internal/webstercli/validate.go:237
    summary: new and changed comments in cards 55-63 break the Comment Line-Break Convention, including a fixed-column wrap in validate.go and unbroken independent clauses in doc.go, planbaseline.go, runlevel.go, tests and the Master stencil
  - id: F10
    severity: NIT
    location: internal/webstercli/validate.go:277
    summary: the plan-edit branch rebuilds findingsEnvelope's JSON by hand instead of passing it the message
  - id: F11
    severity: NIT
    location: internal/websterengine/suspect.go:393
    summary: the recovery plan-path way forward formats the batch as recover-batch %d while every other recover-batch way forward uses %02d
---

The review range is `git diff $(git merge-base main HEAD)..HEAD`, from `product.parent: main` in `_lyx/shed/refusal-audit/status.json`; the merge base is `8cd5b7f60`.
I read cards 55–64, the focus file, and the code each focus entry names, and ran `go build ./...`, `go vet` with no tags, `-tags integration` and `-tags smoke`, and `go test -tags integration` over `internal/websterengine`, `internal/webstercli` and `cmd/lyx`; all pass.

### [BLOCKING] A recovered batch's fork is flagged fork-state-write at run exit

**Location:** `internal/websterengine/runlevel.go:1049` (`forkOwnReport`), with the cause in `internal/websterengine/recoverbatch.go:250` and `internal/websterengine/beginbatch.go:448`

**Issue:** `forkOwnReport` maps a fork transcript to its batch's report through `BatchState.ForkTranscripts`, and falls back to the integration report when no batch holds the transcript.
`recoverSpawn` builds the recovery record from scratch and carries `AuditWarnings` and `SuspectPaths` forward, but not `ForkTranscripts`; a re-begin does the same.
On the ordinary ladder (fork writes a `stuck` report, `record-batch` attributes its transcript, `recover-batch` replaces the record, the recovery lands done, Master reports done), the run-exit audit finds the original fork's transcript in no batch's `ForkTranscripts`.
`CheckFork` then compares its write to `reports/NN-<slug>.md` against the integration report, yields a `fork-state-write` finding that nobody dispositioned, demotes done to stuck and leaves a pending finding whose path is under `_lyx`, which only `run --fresh` clears.
`TestRun_ForkStateWriteAtRunExit` covers only a fork batch that still carries its transcripts.

**Fix:** carry `prior.ForkTranscripts` onto the recovery record in `recoverSpawn` and onto a re-begun record in `BeginBatch`, as both already do for `AuditWarnings`, and add a run-exit regression test over a recovered batch whose original fork wrote its own report.

### [MEDIUM] auditTerminalFork misattributes another fork's transcript to the terminal batch

**Location:** `internal/websterengine/recordbatch.go:412`

**Issue:** every fork in a Master session shares `bs.SessionID`, so `AuditForksIncremental` returns every unseen transcript of the session, not only the terminal batch's.
When `record-batch NN` is repeated on a done batch after batch NN+1 has been begun and its fork has written its report, `auditTerminalFork` checks NN+1's fork with NN's report as `ownReport`.
The legitimate report write becomes `fork-state-write`, batch NN's done record is replaced by a failed one, and `failFromFindings` files NN+1's transcript as seen under batch NN, so `record-batch NN+1` then finds zero new transcripts and archives its report as unattributable.
The repeat call used to be answered harmlessly with "already terminal; continue with the next batch"; it now ends in a `needs_fresh` batch, since the suspect path is under `_lyx`.
The integration fork's unseen transcript is attributed the same way once its report exists.
`runExitAuditCrossCheck`'s per-fork mapping is sound apart from F1: it runs only after `verifyEveryBatchDone`, by which point every batch fork's transcript is attributed.

**Fix:** have `auditTerminalFork` return nil, nil, leaving the already-terminal refusal, when another fork batch of the same session is begun and not terminal, or when the integration report exists, since its unseen transcripts may then belong to that fork.
A forging fork's own transcript is still audited, either here while no later bracket is open or by the next batch's `record-batch`, which sees it as a second new transcript.

### [MEDIUM] run --fresh accepts any HEAD when no recorded start qualifies by ancestry

**Location:** `internal/websterengine/runlevel.go:1155`

**Issue:** card 55 keeps the worktree-HEAD fallback "only when no batch recorded a start and none is missing".
`freshPendingDrop` falls back whenever `bases.Start` is "", which `runEvidenceBases` also returns when starts are recorded but none is an ancestor of all the others (a branch rewritten mid-run).
Then `base == head`, card 58's HEAD rule passes trivially, and the suspect paths are compared against whatever HEAD carries, so the pending findings drop over unaudited commits.
The other callers handle an empty pick safely: `AcceptPendingAudit` reports every tracked path unverifiable toward `--fresh`, and `rebaseline` and `recover-batch` name "the run's start commit" without a SHA.
A missing commit of a kind a caller never uses (a missing head for `--fresh`) still refuses, as both callers' card text requires.

**Fix:** when a start is recorded but none qualifies, require HEAD to be an ancestor of every recorded start, compare the paths against HEAD, and otherwise refuse with `ErrPendingAuditFindings`, naming the recorded starts and a way forward that works: reset the branch to a commit every recorded start descends from (`git merge-base --octopus <starts>`) with git, then re-run `lyx webster run --fresh`.

### [MEDIUM] The run-exit way forward names a git restore for paths nothing can check

**Location:** `internal/websterengine/runlevel.go:1103` (`pendingPathsWayForward`), and `contracts/specs/refusal-spec.md`'s `audit: fork-state-write` row

**Issue:** `pendingPathsWayForward` gives every non-plan path the clause "restore the named paths to the last batch head with git, then run accept-audit".
A `fork-state-write` path is under `_lyx/webster`, and card 63's `.lyx` paths and the scratch directory are git-ignored or untracked, so no git restore applies;
`accept-audit` then refuses them as unverifiable and names `--fresh`, a second refusal before the working route.
The refusal-spec `audit: fork-state-write` row says the run-exit way forward is the `--fresh` route, a clause neither the run-exit stuck reason nor the pending-findings refusal carries for that path.
With only pathless findings, the same function still names "the named paths" when none are named.
Every other card-64 cell I checked matches its message: `missingCommitsClause`, `planPathClause`, the `--fresh` HEAD refusal, `ErrRecoveryNeedsFresh` and `ErrPlanBaselineMissing` (the HEAD-past-head cell is F5).

**Fix:** split the non-plan paths with `trackedRel`: tracked paths keep the git clause, the rest get "reset the branch to the run's start commit with git and run \"lyx webster run --fresh\"" plus the tail, and no path clause is emitted when no path is named.
Update the refusal-spec rows that quote the run-exit way forward (`audit: parent-write, tracked…`, `audit: fabric-reference`, `run-exit audit correctness`, `pending audit findings`) to name the `--fresh` route for a path nothing the run recorded can check.

### [LOW] accept-audit's HEAD-past-head refusal never names accept-audit

**Location:** `internal/websterengine/auditledger.go:125`

**Issue:** `AcceptPendingAudit` wraps `reconcileReportHead`'s refusal unchanged.
That text is written for `record-batch`: "move HEAD back to the report's head_sha … then re-run this verb", and for an unqualified merge "remedy: … redo the parent merge-in after the batch is recorded", which has no `way forward:` clause and names a batch record that accept-audit never makes.
The refusal-spec row `accept-audit: HEAD past last batch head` quotes "re-run `lyx webster accept-audit`", which the message does not contain.

**Fix:** let `reconcileReportHead`'s refusals take the re-run text and the name of the head from the caller, so accept-audit's message reads "move HEAD back to the last batch head <sha> with git, then re-run \"lyx webster accept-audit\"", and give the unqualified-merge refusal a `way forward:` clause.

### [LOW] The recovered-blob search has two gaps

**Location:** `internal/websterengine/suspect.go:409`

**Issue:** the search runs only when `trackedRel` accepts the suspect path at recovery time.
A non-empty `Blob` already proves the path was tracked when the batch failed (`suspectBlobs` records one only then), so the gate only lets a strand that untracks and ignores the original path while moving its content elsewhere skip the search.
Skipping when `headDiff` names the path is harmless, since that path already fails the batch.
When `bs.StartSHA` is empty, `commitBlob` asks git for `:<path>`, which is the index entry, not "absent": after the strand commits, the index holds head's blob, so every moved copy reads as held at the start and is skipped.
For a path the start commit lacks, `commitBlob` correctly returns "".

**Fix:** drop the `trackedRel` gate, and treat an empty `StartSHA` as a start that holds no path.

### [LOW] gitwrap.go's header and the raw-git guard reason are stale

**Location:** `internal/websterengine/gitwrap.go:1`, `cmd/lyx/rawgitmutation_test.go:46`

**Issue:** every git probe the evidence-base code adds (`shaExists`, `isAncestor`, `treePathsWithBlob`, `headSHA`) lives in `gitwrap.go`, and no direct `gitrepo` or `gitexec` use remains elsewhere in the package.
But the header lists the suspect-path probes without `treePathsWithBlob`, and says every helper goes through `gitrepo.Repo` except `dirty` and those probes, which `treePathsWithBlob`'s `gitexec.Run` contradicts.
The guard's allowlist reason, which enumerates each read-only command, omits `ls-tree -r` and the `SHAExists`/`IsAncestor` uses of `gitrepo.New`.
The guard is a file-level allowlist, so it still passes, but its recorded reason no longer covers the file.

**Fix:** name `treePathsWithBlob`, `shaExists` and `isAncestor` in the header, and `ls-tree -r` (treePathsWithBlob) and `SHAExists`/`IsAncestor` in the allowlist reason.

### [LOW] storePlanBaseline can store a wrong copy

**Location:** `internal/websterengine/planbaseline.go:48`

**Issue:** `storePlanBaseline` writes the file's current bytes to the file named by a hash its caller computed earlier: `Run` computes `fileHashes` at entry and stores them only after the strand reclaim and `freshPendingDrop`.
A plan edited in between stores bytes that do not hash to their name, and since an existing copy is never rewritten, that wrong copy stays.
`RestorePlan` checks the hash and reports the file missing, so nothing is restored wrongly, but the doc's "no stored copy is ever wrong" is false and the copy is lost for good.

**Fix:** hash the bytes read and return an error when they do not match the recorded hash, writing nothing for that file.

### [LOW] Comment and Markdown line breaks

**Location:** `internal/webstercli/validate.go:237`, `internal/websterengine/doc.go:154`, `internal/websterengine/planbaseline.go:1`, `internal/websterengine/runlevel.go:1120`, the test doc comments listed below, and `contracts/stencils/webster/webster-template-master.md:146`

**Issue:** CONSTRAINTS.md's Comment Line-Break Convention requires a break at a semicolon or a comma plus "and"/"or"/"but" that starts an independent clause, and no fixed-column wrapping in new comments.
- `validate.go:237-239` is a new comment wrapped at a fixed column.
- `doc.go`: "A refusal there mutates nothing; validate still lints…", "…replaces the terminal record with a failed one, and otherwise the \"already terminal\" refusal stands.", "…before its first rewrite, and record-batch and recovery also check…", "…before spawning anything, and the way forward is…", "…dropped with the archived state, and its warning says so."
- `planbaseline.go`: the header's "…records only hashes, and a plan directory is not tracked…", "An absent file is false with no error; any other stat failure is returned.", "…after checking the stored bytes hash to the recorded value, and a file the run never recorded is removed.", "Every needed copy is checked before anything is written; a missing or corrupt copy refuses…".
- `runlevel.go`: `freshPendingDrop`'s "…since no verb could restore the last; its warning says so.", "The start commit is picked by git ancestry, and a recorded commit missing…", "…join the path check, and it adds its own drop warning."; `pendingPathsWayForward`'s and `pendingAuditFindingsError`'s "…, and the git clause covers only the other paths."
- Tests: `TestBatchCardEditError_NamesTheEditedBegunCard`, `TestRun_FreshDropsPlanPathWithoutCopy`, `TestValidateCmd_RefusesOverviewEditWithoutRestamp` and the fabric-reference uncheckable test in `recordbatch_test.go` each end in ", and <independent clause>".
- The Master stencil's "…or since the batch began, and the same rung applies."
The `contracts/specs` edits follow the rule.

**Fix:** break each at the clause boundary, and rewrap `validate.go:237-239` one sentence per line.

### [NIT] validate's plan-edit envelope duplicates findingsEnvelope

**Location:** `internal/webstercli/validate.go:277`

**Issue:** the plan-edit branch hand-builds the same `ok`/`error`/`findings`/`scope` JSON as `findingsEnvelope`, differing only in the message.

**Fix:** give `findingsEnvelope` the message as a parameter and call it from both branches.

### [NIT] The plan-path way forward formats the batch number unpadded

**Location:** `internal/websterengine/suspect.go:393`

**Issue:** `checkRecoveredSuspects` names `"lyx webster recover-batch %d"`, while `RecordBatch`, `RecoverSpawnOrAttach` and the other recover-batch way forwards use `%02d`.

**Fix:** use `%02d`.

## Focus entries settled as not a defect

- Card 57: `storePlanBaseline` is content-addressed and writes through a temp file and rename (F8 aside); `RestorePlan` checks every needed copy and its hash before writing or removing anything; every restamp site and both `runlevel.go` initialisation sites store before mutating or saving state; `restore-plan` holds the state-mutation lease and reports a sync failure with `fabricSyncWayForward`.
  Its Create group duplicates no plan-restore or content-store helper; `writeFileAtomic` overlaps the generic `fsx.AtomicWriteBytes`, which leaves the file mode at 0600, and restoring a card should keep 0644.
- Card 58: the refusal order (differing tracked paths, then HEAD, then plan paths) happens wholly before archiving, and `pendingAuditFindingsError` and the run-exit stuck reason split plan paths from git paths as the card specifies (F3 and F4 aside).
- Card 59: `PlanEditError` and `batchCardEditError` run after the correctness branch and before attribution in `RecordBatch`, and after `checkRecoveredSuspects` and before `postBatchChecks` in `PersistRecoveryTerminal`, with no mutation before them; validate lints, skips the restamp and exits non-zero with the findings; both verbs emit `plan_drifted`.
- Card 60: `failOnCorrectness` serves both the main branch and `auditTerminalFork`, so both record `Uncheckable`; `RecoverSpawnOrAttach` refuses before any spawn or mutation; the envelope and stencil rung match the error.
- Card 63: `filepath.Dir(geom.ScratchDir)` is `<anchor>/.lyx` in hub geometry and `<state>/.lyx` in standalone geometry, the run's `.lyx` state directory in both, and neither narrower nor broader.
- Card 56: checking suspect paths against the reconciled HEAD rather than the last batch head only admits content a clean parent merge brought, which the reconcile already deems legitimate.
- Edit groups: every changed signature (`AcceptPendingAudit`, `CheckFork`, `restampFingerprint`, `RestampPlanBaseline`, `failFromFindings`, `RebaselineDeps.Geom`, `failBatchInput.Uncheckable`) has every caller updated; `go vet` passes under no tags, `integration` and `smoke`.
