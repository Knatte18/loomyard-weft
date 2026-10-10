---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md#D1, Open risks, Acceptance criteria
    summary: D1's deadline bound ends a held gated run OutcomeDone, not OutcomeTimeout, since its output files exist; the Open risk and acceptance criterion require an outcome PATTERN-completion-signal forbids
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md#D6
    summary: D6's environment set, mentioned GOOS plus one unmentioned, misses the GOOS-implied unix tag, so the classifier either fails open on !windows beside unix or fails the unix-beside-_windows.go criterion
  - id: F3
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md#D3
    summary: D3's contract-file bound rests on a false premise; the evidence reads Master and fork writes, and a recovery slice it never reads lets a recovery's later outcome.yaml write clear as Master's
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md#D3
    summary: Widening the reset's own-path set exempts a recovery's tracked writes from the reset dirt check, and D3 states neither what a reset may now discard nor a criterion for it
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md#D2
    summary: D2 makes the re-prompt a per-entry one-line text, but the line must carry the per-evaluation findings path, which an entry cannot know at wiring time
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md#Scope item 5
    summary: Scope item 5 carries probe results and issue history that belong in support-log.md, where interview question 7 already records them
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md#Acceptance criteria
    summary: D6's Rename-new done-check and the plan-gate member reads have no acceptance criterion, though Scope item 6 and D6 change both
---

### [BLOCKING] D1's deadline bound does not end the run `OutcomeTimeout`

**Location:** `decision-record.md` D1, the first Open risk, the second D1 acceptance criterion.

**Issue:** D1 holds a gated run whose output files exist, and bounds the hold by the run deadline.
The deadline exit in `internal/shuttleengine/wait.go` goes through `classifyDeadlineExpiry(OutcomeTimeout)`, which returns `OutcomeDone` whenever every output file exists, and the `handleGatedBoundary` doc says that path then runs the gate once more through finalize.
D1's scenario has every output file present by definition, so a shell whose completion notification never arrives ends the run `done` at the deadline, with the gate evaluated while the shell may still be running.
The Open risk ("the run then ends `OutcomeTimeout`") and the criterion ("ends `OutcomeTimeout` with the shell in `EndedShells`") are false against the code.
Meeting the criterion would mean finalizing a negative verdict while the file contract is satisfied, which `PATTERN-completion-signal` forbids, a BLOCKING constraint breach.
D1 also never states this outcome: after the deadline, the run reads `done` with the shell's gate tier unread, the fault the Goal sets out to close, now delayed rather than removed.

**Fix:** State in D1 what the deadline bound yields: `OutcomeDone` through `classifyDeadlineExpiry`, the gate evaluated once at finalize, and the shell recorded in `EndedShells` with its result unread.
Correct the Open risk and the acceptance criterion to that outcome, and say why a held shell past the deadline is acceptable, or decide a different bound that keeps to `PATTERN-completion-signal`.

### [BLOCKING] D6's environment enumeration misses the `unix` tag

**Location:** `decision-record.md` D6, its second paragraph's environment set; the acceptance criterion on `//go:build unix` beside `_windows.go`.

**Issue:** D6 enumerates GOOS over "the OS names the candidates mention plus one they do not".
`go/build.Context.matchTag` makes `unix` true for every GOOS in its Unix list and makes `linux`, `solaris` and `darwin` true under `android`, `illumos` and `ios`, so the unmentioned OSes are not interchangeable.
Take `//go:build !windows` beside `//go:build unix`: they overlap on `linux`.
If `unix` counts as an OS-derived tag, the set is {`windows`, one unmentioned GOOS}; with `plan9` as the unmentioned one, no environment selects both, and the classifier passes a genuine ambiguity, which fails open on the guard D6 weakens.
If `unix` instead counts among "the other tags" and is assigned freely through `BuildTags`, the assignment `unix` true under `GOOS=windows` selects both files of the criterion's `//go:build unix` and `_windows.go` pair, so that criterion fails.
Either reading breaks D6's stated bound that "only an answer whose candidates provably never coexist passes".

**Fix:** Enumerate every GOOS and GOARCH in the closed list planglyph already pins, rather than the mentioned names plus one, and state that `unix` and the GOOS implications are derived from GOOS, never assigned as free tags.
Add an acceptance row for `!windows` beside `unix` staying `glyph-ambiguous`.

### [BLOCKING] D3's contract-file bound rests on a false premise

**Location:** `decision-record.md` D3, "The contract-file evidence keeps reading the Master slice alone" and its Bound sentence.

**Issue:** `contractFileStatus` in `internal/websterengine/contractevidence.go` reads two slices: it clears a contract path when the latest successful Master write is later than every fork write.
It does not read the Master slice alone, and fork writes are what keep a finding standing.
A recovery slice that the evidence never reads is invisible to it: a recovery that writes `outcome.yaml` after Master's last write leaves that Master write as the latest one the check sees, so the finding clears as Master's rewrite.
That is the exact case D3's bound and its acceptance criterion say stays a finding.
This is a weakened guard whose stated bound does not hold as specified.

**Fix:** Decide that the contract-file evidence counts recovery writes on the fork side, so a recovery write later than Master's latest one keeps the finding.
Correct the sentence to name both slices the evidence reads and add the recovery slice to the side that keeps the finding standing.
Pin it with an acceptance row: a recovery write to `outcome.yaml` after Master's last write leaves the finding standing.

### [MEDIUM] Widening the reset's own-path set has no stated bound

**Location:** `decision-record.md` D3, "`writtenWorktreePaths` and the reset's own-path set read all three slices".

**Issue:** The reset's own paths (`ownTrackedPaths`, `ResetDeps.OwnPaths`) are the tracked paths "exempt from the dirt check".
Adding recovery writes to them means a reset now discards a recovery's uncommitted tracked edits without the dirt refusal it gives today.
D3's Bound sentence says recovery writes join the own-path set but not what that lets a reset skip, and no acceptance criterion covers the reset side.

**Fix:** State in D3 that a reset now discards, without refusal, tracked paths a recovery session of this run wrote successfully, and that the exemption stays by path and on successful writes only.
Add an acceptance criterion for a reset over a recovery-written tracked path.

### [LOW] D2's per-entry text cannot carry the findings path

**Location:** `decision-record.md` D2.

**Issue:** `gateRepromptText(findingsPath)` in `internal/shuttleengine/gate.go` builds the line from a path known at each failed evaluation.
D2 makes the replacement "an optional one-line re-prompt text an entry supplies", yet requires it to open with `Gate findings recorded at <path>`.
A text fixed at wiring time cannot hold that path, so the decision leaves open how the entry's line gets it.

**Fix:** Say the entry supplies a line rendered with the findings path, for example a function of the path, validated by the same one-line rule at send time.

### [LOW] Scope item 5 carries probe history

**Location:** `decision-record.md` Scope item 5, its second and third sentences.

**Issue:** The probe results at HEAD and "#511 is #500 in another coat" are deliberation narrative; `support-log.md` interview question 7 already records them.
Completeness test: the one requirement they carry, no resolver change, is already stated in D5's own Decision.

**Fix:** Cut those two sentences from Scope item 5 and keep its first sentence; leave D5's "No resolver change" with its reason in one clause.

### [LOW] D6 behaviors without an acceptance criterion

**Location:** `decision-record.md` Acceptance criteria, against D6 and Scope item 6.

**Issue:** D6 changes the Create done-check and the Rename-new done-check direction, and the plan-gate passes that resolve a member into its file.
The criteria name Edit, Delete, Rename-old, `Uses:` and the Create done-check, but no criterion covers the Rename-new done-check or a plan-gate pass over a partitioned answer.

**Fix:** Extend the partitioned-member criterion to the Rename-new done-check and to a plan-gate member read over a partitioned answer.

## Verdicts on the focus entries without a finding

- D6 against Scope item 6 on Create: the two agree.
  Scope item 6 speaks of the Create done-check after the card (`DoneChecks`), and D6 and the criterion keep `createFindings` at Plan time as `create-already-exists`; these are two different checks, and the support log's rejected alternative states why.
- D3's non-zero exit for `stuck` and `dead`: bounded.
  It removes no guard; D3 says Master reads digest fields rather than the exit code, and the Open risks name the script-facing change.
- Evidence slices across D3, Constraints and Acceptance criteria: the written-path side agrees; the contract-file and reset sides are F3 and F4.
