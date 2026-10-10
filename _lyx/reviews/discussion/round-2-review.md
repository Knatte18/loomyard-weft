---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md#D7
    origin: handler
    summary: D7 guard refuses push under a webster strand name, which also refuses the detached push child that every fabric commit from that strand spawns, silently dropping the records push
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md#D7
    origin: lens:generic
    summary: D7 refuses every non-reader fabric verb for task roles while refusal-spec rows still name lyx fabric commit and push as the way forward for those roles
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md#D7
    origin: lens:error-handling
    summary: D7 states no handling for the inherited strand name in gate test runs and in lyx config nested fabric sync, so fabric tests and per-worktree config writes from a slugged role would be refused
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md#D12
    origin: lens:generic
    summary: D12 cap of two binds every caller including Master's card_amended re-run, but the Decision neither counts nor exempts it and adds no Master rung for the new recovery_exhausted refusal
  - id: F5
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md#D12
    origin: lens:correctness
    summary: D12 names only recoverSpawn, rebaseline and reset for the new counters, but BeginBatch rebuilds the batch record and carries over only StartSHA and two lists, so a re-begin resets Recoveries and RecoveryStartSHA
  - id: F6
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md#D9
    origin: lens:generic
    summary: D9 gated exception keys on a gate re-prompt having been sent, but a PassOnCap pending send and a reattached run both leave an earlier arrival with no re-prompt counted in memory
  - id: F7
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md#D9
    origin: lens:test-gaps
    summary: D9 says a gated run before any re-prompt counts a files-present turn end at once, contradicting the existing gated-waiting-with-files-present test, and names TestTurnEnded as holding expiry rows it does not have
  - id: F8
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md#D8
    origin: lens:generic
    summary: D8 has ClassifyViolation and RecoverSpawnOrAttach call the fabricengine classifier, but websterengine production code imports no fabricengine and takes the classifier by injection; the Decision does not say which
  - id: F9
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md#D8
    origin: lens:test-gaps
    summary: D8 git read forms use exact-word denylists, so git option abbreviations and a help word consumed as a flag value pass the classifier, contradicting its bound that nothing outside the listed forms is admitted
  - id: F10
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md#D1
    origin: lens:error-handling
    summary: D1 blocks the row with Rebaseline's own refusal text, whose way forward names lyx webster run and a manual rebaseline, wrong for a loom row that re-steps and would have rebaselined itself
  - id: F11
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md#D8
    origin: lens:correctness
    summary: D8 says needs_fresh is reached only for a rejected command, yet the in-line accept still fails on the evidence rule, and the existing bound that no listed reader can write is not restated for git status and lyx forms
  - id: F12
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md#D9
    origin: lens:error-handling
    summary: D9 leaves open whether an outstanding fork or awaited-prefix shell holds a turn end when every output file is present
  - id: F13
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md#D12
    origin: lens:correctness
    summary: D12 recovery_retry is HEAD past RecoveryStartSHA with no rule for an empty RecoveryStartSHA on older records, a HEAD moved by merge-in or reset, or a started strand whose record save failed
  - id: F14
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md#D1
    origin: lens:error-handling
    summary: D1 maps only the fingerprint and card-set sentinels to Stuck and leaves open the other Rebaseline errors, which would surface as producer-hard-failure with a way forward naming the wrong command
  - id: F15
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md#D5
    origin: lens:correctness
    summary: D5 splitter does not say a backslash-newline continuation joins the line, and splitting at pipe and double pipe can carry a cd from a pipeline stage into later segments of crossModuleCommands
  - id: F16
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md#D2
    origin: lens:generic
    summary: D2 adds the webster_record marker to the Plan-Burler fixer prompt, but that prompt has no rubric markers and the fixer already receives the rubric, so the separate marker rests on a false premise
  - id: F17
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md#D3
    origin: lens:generic
    summary: D3 names two causes for the empty frame but recordedOverviewFrame also returns empty when the recorded overview hash is empty, and the cause must reach the refusal through a changed return
  - id: F18
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md#D13
    origin: lens:error-handling
    summary: D13 pattern-entry-line-cap finding names entry and length but no way forward, and the inline-span branch cannot match a line that itself contains backticks
  - id: F19
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md#Acceptance criteria
    origin: lens:test-gaps
    summary: Acceptance criteria lack the unedited done card for D2, the Stuck reason opening with the rebaseline warning for D1, the D11 no-reproduction branch, a files-present gated case for D9, and D12 cap, reset-to-start and rebaseline-keeps-count cases
---

### [BLOCKING] D7 guard refuses the detached push child

**Location:** `_lyx/discussion/decision-record.md` D7, second part, and the acceptance criterion on `push`.
**Issue:** D7 refuses every fabric subcommand but the readers from a slugged strand whose role is not driver, orch or conflict.
`Fabric.Commit` (`internal/fabricengine/commit.go`) spawns `SpawnDetachedPush` (`internal/fabricengine/spawn.go`) whenever anything landed, and that call sets no `Env`, so the child `lyx fabric --warp-path … --weft-path … push` inherits the strand name.
`webstercli`'s `fabricSync` (`internal/webstercli/sync.go`) calls `Fabric.Commit` from the Master strand, whose role is `webster`, so every bracket verb's records push would be refused, silently, because the child is fire-and-forget.
That breaks PATTERN-push-both-sides for the webster strand and any other slugged role that commits through lyx.
D7's Bound covers only unset, empty and unparseable names; the bypass-mode flags are never mentioned, and the acceptance criterion reads as requiring the refusal for `push` under a webster name.
**Fix:** state in D7 that an invocation carrying the injected `--weft-path` or `--warp-path` bypass is outside the guard, with what bounds that (the flags are hidden and only Go sets them, so a fork can still use them; say so in the Bound), and add an acceptance row that a bypass-mode push runs under a webster strand name.

### [MEDIUM] D7 contradicts refusal-spec ways forward

**Location:** `_lyx/discussion/decision-record.md` D7; `contracts/specs/refusal-spec.md` rows `fabric sync failed`, `decision: commit failed after the append`, `commit-records: push rejected`, `commit-records: push failed`.
**Issue:** Those rows name `lyx fabric commit` or `lyx fabric push` as the step for roles that D7 now refuses (the Master sees `fabric sync failed`, a loom session sees the decision-commit row).
D7 reconciles only its own refusal with PATTERN-refusal-way-forward.
**Fix:** state in D7 which of the two changes: the spec rows' way forward moves to the orch, or the verb set the guard refuses excludes `commit` and `push`.

### [MEDIUM] D7 has no stated handling of the inherited strand name in tests and nested calls

**Location:** `_lyx/discussion/decision-record.md` D7.
**Issue:** `gateslot.Lease.Env` (`internal/gateslot/pool.go`) copies the base environment, so `lyx gate test` run by an implementer or recovery strand carries its strand name into fabric-verb tests that go through the cobra pre-run; the strand D7 targets is the one that runs them for its card gates.
`configcli` already has to clear the variable in its integration test for the same reason.
`lyx config` also calls `fabriccli.RunCLIIn(cwd, w, []string{"sync"})` after a per-worktree write (`internal/configcli/configcli.go`), so under D7 the file is written and the sync refused.
**Fix:** state that tests set or clear the name through the test kit, and say whether the nested config sync is exempt or moves to the orch.

### [MEDIUM] D12 cap against Master's card_amended re-run

**Location:** `_lyx/discussion/decision-record.md` D12; `contracts/stencils/webster/webster-template-master.md` card_amended rung.
**Issue:** The cap binds every caller, and Master re-runs `recover-batch` once per amendment, even after a failed recovery.
Recovery, a `card_amended` re-run and a dead-recovery retry can exhaust the cap, after which the next amendment meets `recovery_exhausted`.
D12 changes only the dead rung; the ladder has no rung for the new refusal, and D1's crash-amendment also lands as a re-run.
**Fix:** state whether `card_amended` re-runs count toward `Recoveries` or reset it, and add a Master rung for `recovery_exhausted` (write `outcome: stuck` quoting the refusal), with an acceptance row.

### [MEDIUM] D12 counters lost on re-begin

**Location:** `_lyx/discussion/decision-record.md` D12; `internal/websterengine/beginbatch.go`.
**Issue:** `BeginBatch` builds the batch record anew and carries over `StartSHA`, `AuditWarnings` and `ForkTranscripts` only, so Master's `report_archived` re-begin resets `Recoveries` and `RecoveryStartSHA` to zero.
That defeats "at most two recoveries per batch for every caller".
**Fix:** add `Recoveries` and `RecoveryStartSHA` to what a re-begin keeps, beside the rebaseline and reset rules.

### [MEDIUM] D9 gated-run exception keys on the wrong event

**Location:** `_lyx/discussion/decision-record.md` D9.
**Issue:** "After it has sent a gate re-prompt" is tracked by `gateSent` and `gateFails`, which are in-memory and zero on a reattached run.
A `PassOnCap` pending send is also an earlier arrival with no re-prompt.
In both cases leftover output files would count a waiting turn end at once, the failure the exception exists to prevent.
**Fix:** reword the exception to any arrival the gate already handled, and say how a reattached run derives it.

### [MEDIUM] D9 test claims disagree with the tests

**Location:** `_lyx/discussion/decision-record.md` D9 and Acceptance criteria; `internal/shuttleengine/waiting_test.go`, `internal/websterengine/strand_test.go`.
**Issue:** `TestPollEventsTick_GatedWaitingWithOutputFilesIsNotAnArrival` pins that a gated run with files present and a waiting turn end, before any re-prompt, is not an arrival; D9 says the opposite and does not list the test as reversed.
`TestTurnEnded` has no shell or outstanding rows; the expiry rows are in `waiting_test.go` and `TestRun_ExpiredShells` in `runlevel_test.go`, and the `TurnEndedAfter` outstanding rows are new.
**Fix:** state that the gated test flips or stands, name the real tests to reverse, and say the recovery-strand rows are new.

### [MEDIUM] D8 classifier reaches websterengine without a stated seam

**Location:** `_lyx/discussion/decision-record.md` D8; `internal/websterengine/doc.go`, `internal/websterengine/auditledger.go`.
**Issue:** `websterengine` production code imports no `fabricengine`; `AcceptBatchFabricReference` takes the classifier as an argument that `webstercli` supplies.
D8 has `ClassifyViolation` and `RecoverSpawnOrAttach` use it without saying whether it is injected, which changes both signatures, or imported, which breaks the package's documented invariant.
**Fix:** state the injection and which signatures change.

### [MEDIUM] D8 git read forms are denylists under an allowlist bound

**Location:** `_lyx/discussion/decision-record.md` D8 and the acceptance classifier list.
**Issue:** `ls-remote`, `log`, `show`, `diff`, `status` and `rev-parse` reject exact flag words, but git accepts unambiguous long-option abbreviations (`--upload-pa=`, `--out=`), and `ls-remote` also takes `--exec` as a synonym.
A help word consumed as a flag value (`lyx fabric add --branch --help`) passes the help rule and runs the verb.
The Bound says the classifier admits nothing outside the forms above.
**Fix:** use a positive flag allowlist per form, as the `branch`, `cat-file` and `config` forms already do, restrict help acceptance to a position the parser can prove, and add these rejects to the acceptance list.

### [MEDIUM] D1 Stuck reason carries a wrong way forward for a loom row

**Location:** `_lyx/discussion/decision-record.md` D1; `internal/websterengine/rebaseline.go`, `internal/websterengine/runlevel.go`.
**Issue:** Rebaseline's refusals build their steps from `stepRun` (`lyx webster run`) and tell the operator to rebaseline the card by hand, while Run threads `deps.reentryStep()` so a loom hub is told to re-step.
D1 keeps that text as the Stuck reason, so the loom row blocks with steps that name the wrong command, against the Constraints' own PATTERN-refusal-way-forward bullet.
**Fix:** state that Rebaseline takes the row's re-entry step and that the auto path's refusal says to re-step after fixing the plan.

### [LOW] D8 acceptance wording and existing evidence bound

**Location:** `_lyx/discussion/decision-record.md` D8; `internal/websterengine/auditledger.go`.
**Issue:** "needs_fresh is reached only for a command the classifier rejects" ignores the evidence rule (HEAD at the start or tree clean), which still refuses.
`AcceptBatchFabricReference` rests on "no listed reader can write", which `git status` and the `lyx` forms change.
**Fix:** name the refusal the in-line path returns when the evidence fails and say the bound's doc is rewritten.

### [LOW] D9 forks and awaited shells with files present

**Location:** `_lyx/discussion/decision-record.md` D9 and the shell criterion.
**Issue:** Whether a fork or an awaited-prefix shell holds a turn end when every output file is present is unstated; the criterion "keep waiting" is not conditioned on missing files.
**Fix:** state that they hold only when a file is missing, or that they hold always, and condition the criterion.

### [LOW] D12 recovery_retry predicate

**Location:** `_lyx/discussion/decision-record.md` D12.
**Issue:** No rule for an empty `RecoveryStartSHA` on older records, for a HEAD moved by merge-in or reset rather than by the recovery's commits, or for a started strand whose record save fails and so is not counted.
**Fix:** define "past" as the recovery's own commits, make an empty start false, and say how an unrecorded started strand counts.

### [LOW] D1 other Rebaseline errors

**Location:** `_lyx/discussion/decision-record.md` D1.
**Issue:** Rebaseline's transient read and restamp errors and a nil plan are not mapped, so they would be returned errors and end the run as `producer-hard-failure` with a way forward naming `lyx webster rebaseline`.
**Fix:** say which of these are Stuck and which stay errors.

### [LOW] D5 splitter edges

**Location:** `_lyx/discussion/decision-record.md` D5.
**Issue:** A backslash-newline continuation would split `go test \` from its packages; `crossModuleCommands` carries a `cd` across segments, and splitting at `|` and `||` lets a `cd` in a pipeline stage or after `||` persist into later segments.
**Fix:** state the continuation join and that a `cd` binds only the `&&`, `;` and newline sequence.

### [LOW] D2 fixer-prompt marker premise

**Location:** `_lyx/discussion/decision-record.md` D2 first part; `contracts/stencils/burler/burler-step-3-fix.md`, `internal/shedadapters/rubric.go`.
**Issue:** `publish_failure` is filled in the rubric reader only; the shared fixer step has no such marker and receives the rubric through the explore step.
A marker on the Plan-Review rubric already reaches the fixer, so the separate fixer-prompt marker would put a segment input into the generic burler composer for nothing.
**Fix:** limit the marker to the rubric and drop `burler-step-3-fix.md` from the Notes.

### [LOW] D3 third empty-frame cause

**Location:** `_lyx/discussion/decision-record.md` D3; `internal/websterengine/rebaseline.go`.
**Issue:** `recordedOverviewFrame` also returns empty when the recorded overview hash is empty, and it returns only `(string, error)`, so the cause cannot reach `overviewRefusal`.
**Fix:** name all three causes and the changed return.

### [LOW] D13 finding without a way forward

**Location:** `_lyx/discussion/decision-record.md` D13 and Constraints.
**Issue:** `pattern-entry-line-cap` names entry and length only, and a line starting with ``- `PATTERN-`` contains backticks, so the inline-span branch cannot match it.
**Fix:** add the shorten-or-move-to-prose way forward and drop the inline-span clause.

### [LOW] Acceptance criteria gaps

**Location:** `_lyx/discussion/decision-record.md` Acceptance criteria.
**Issue:** Missing: an unedited done card for `done-card-edited`; the Stuck reason opening with the rebaseline warning; the D11 no-reproduction branch; a gated files-present case with no shell; for D12, `recovery_retry` false at two recoveries, `reset --to start` clearing counts and a rebaseline keeping them.
**Fix:** add these as rows or assertions of the existing scenarios.

## Rejected

- Implementation pointers in D8 and D9 (test rows flipped, doc comment rewritten) belong in Notes: kept, since they tell the plan writer what existing tests change.
- `Result.ExpiredShells` keeps a name that no longer says expired: a naming choice for the plan writer.
- `TurnEndedAfter` ignoring a session-signal parse error: an implementation detail for the plan writer's code, not a Decision.
- Missing test row for a prefix naming no objects in D4: covered by the existing empty-result behavior and the enumeration rule of the rubric.

Prior-round files were not read before this review was saved.
