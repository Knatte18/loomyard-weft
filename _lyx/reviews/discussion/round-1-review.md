---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md:47
    summary: The half-broken-pair fallback contradicts the refuse-on-error decision and, for an unopenable pair, would let start spawn on an unchecked tree
  - id: F2
    severity: LOW
    location: _lyx/discussion/decision-record.md:48
    summary: Rejecting Fabric.MergeInProgress for needing an open Fabric is not a real obstacle, since fabricengine.Open(l) is the public constructor and the new probe builds its own handle from l
  - id: F3
    severity: LOW
    location: _lyx/discussion/decision-record.md:76
    summary: The batten retry behaviour is verifiable in the code and is verified non-retryable, yet it stays an assumption, an open risk and open ledger Q7
  - id: F4
    severity: LOW
    location: _lyx/discussion/decision-record.md:35
    summary: The record never states what start does on a mid-merge pair whose live driver has halted at a hand-back without parking, leaving the precedence between driver_not_parked and merge_in_progress to the plan writer
  - id: F5
    severity: LOW
    location: _lyx/discussion/decision-record.md:45
    summary: The form of the reported conflicted paths is unspecified, though the refusal tells the operator to pass them to lyx fabric merge-stage
  - id: F6
    severity: NIT
    location: _lyx/discussion/decision-record.md:7
    summary: The Goal sentence describing today's bug reads as the desired behaviour
---

### [MEDIUM] Half-broken-pair fallback contradicts refuse-on-error

**Location:** `_lyx/discussion/decision-record.md:47`, against `:59`.

**Issue:** The probe decision says a pair whose record location cannot be resolved "follows `mergeBlocksMutation`'s half-broken-pair rule and reports on the git-level probes alone".
`mergeBlocksMutation` (`internal/fabricengine/mergestate.go`) covers two cases and answers `(false, nil)` for both: `newPaired` failing on a missing warp or weft directory, and `weftGitDir` failing.
In the first case there is no `*Fabric`, so the git-level probes cannot run at all; following the rule literally returns "not mid-merge" and `start` spawns.
In the second case `weftGitDir` fails exactly when `git rev-parse --git-dir` fails in the weft path (`internal/fabricengine/index.go`), and `MergeHeadPresent` runs `git rev-parse` in the same directory, so the weft git-level probes fail too and the refuse-on-error decision takes over.
The fallback is therefore either unsafe (unopenable pair) or dead (unresolvable gitdir), and the two decisions give opposite answers for an unopenable pair.
Its rationale also does not transfer: `mergeBlocksMutation` answers false so a half-torn-down pair can still be torn down, while `start` is about to put a driver on the pair, where "undetermined is not clean" is the record's own stated rule.

**Fix:** Drop the half-broken-pair sentence.
State instead that a pair the probe cannot open, or whose record location cannot be resolved, is a probe error, which `start` refuses on per the probe-error decision.
Add the unopenable-pair case to the probe-error acceptance criterion.

### [LOW] Inaccurate rationale for not reusing Fabric.MergeInProgress

**Location:** `_lyx/discussion/decision-record.md:48`; the same claim in `support-log.md` Interview item 5 and Rejected alternatives.

**Issue:** The rationale rejects `Fabric.MergeInProgress` as "record-only, needs an open `*Fabric`".
`fabricengine.Open(l)` (`internal/fabricengine/open.go`) is the public, `l`-in constructor every outside package calls, so `loomcli` could hold an open `*Fabric` without difficulty.
The new probe itself reuses `mergeRecordExists` and `foreignMergeStatePresent`, both methods on `*Fabric`, so it must open the pair from `l` internally anyway.
The decided shape is consistent and actionable; only the "needs an open `*Fabric`" half of the rationale is wrong, and it invites the plan writer to think opening a handle is something the probe must avoid.

**Fix:** Keep "record-only" as the reason, drop "needs an open `*Fabric`", and state that the probe opens the pair from `l` itself.
Correct the matching support-log lines.

### [LOW] Batten's handling is verified, not an assumption or a risk

**Location:** `_lyx/discussion/decision-record.md:76` and `:83`; `support-log.md` Question ledger Q7 and Interview item 8.

**Issue:** The record states batten's non-retryable handling as an auto-mode assumption, repeats it as an open risk that the loop "must" be confirmed by the plan, and the support log leaves Q7 open.
The three agree in content but all hedge on a fact the codebase answers.
`internal/battencli/wire.go`'s `childSpawnError` wraps `battenshed.ErrChildNotParked` only when `childNotParked` finds kind `driver_not_parked`; any other refusal returns a plain error.
`internal/battenshed/innerrun.go` retries only on `ErrChildNotParked` and returns every other spawn error as a hard error, so a `merge_in_progress` refusal cannot loop.
An unverified loop risk handed to the plan writer costs a verification pass and suggests the plan may need a batten change that the Scope already rules out.

**Fix:** Move the verified fact into the kind decision's rationale: a `merge_in_progress` refusal reaches batten as a hard spawn error, not a retry, so batten needs no change.
Delete the batten auto-mode assumption and the batten open risk, and resolve Q7 in the support log with the evidence.

### [LOW] Precedence against the not-parked refusal is unstated

**Location:** `_lyx/discussion/decision-record.md:35`–`38`; Acceptance criteria.

**Issue:** `runDriverSpawnAndWait` has three outcomes for a live strand: resume (marker present), working (no marker, running run) and refused as `driver_not_parked` (no marker, run halted at a hand-back).
The record names the first two for a mid-merge pair but not the third.
The trigger decision implies the third is neither a spawn nor a resume, so `start` keeps refusing with `driver_not_parked`, batten retries, and the resume after the park then meets `merge_in_progress`.
A plan writer who places the check ahead of the whole live-strand branch would instead return `merge_in_progress` there; either order is defensible, so the record has to pick.

**Fix:** State in the trigger decision that a live strand with no marker over a run halted at a hand-back keeps its `driver_not_parked` refusal, and the merge check applies once the driver parks and `start` would resume it.
Add a matching acceptance criterion.

### [LOW] Conflicted-path form unspecified

**Location:** `_lyx/discussion/decision-record.md:45` and `:51`.

**Issue:** The probe returns "the conflicted paths" and the envelope carries them in `conflicts`, while the fabric-parked message tells the operator to run `lyx fabric merge-stage <path>...` on them.
A two-sided probe gathers unmerged entries from two checkouts; concatenating the raw per-side lists would report weft paths relative to a checkout the operator never sees, in a form `merge-stage` does not accept.
The fabric merge verbs already report conflicts in the pair's one-repo form (`MergeResult.Conflicts`), which is what the remedy expects.

**Fix:** State that the probe reports conflicted paths in the same one-repo form a fabric merge verb's `conflicts` carries.

### [NIT] Goal sentence reads as the fix

**Location:** `_lyx/discussion/decision-record.md:7`.

**Issue:** "`start` checks for no merge state at all before it spawns or resumes a driver" sits between the incident and the fix, and reads as "`start` checks that there is no merge state", which is the desired behaviour, not today's gap.

**Fix:** Reword as "Today `start` checks no merge state at all before it spawns or resumes a driver".

## Focus verdicts without a finding

- Live-driver exemption and the crash-left Landing record (focus 5): not a defect.
  `internal/mergeresolve` runs inside a step with a live driver, aborts on every non-concluding exit, and aborts a leftover fabric record on re-entry (`mergeresolve.go`, the `MergeInProgress` then `MergeAbort` opening), so both the rationale and the auto-mode assumption's parenthetical hold as written.
  The open risk carries a real operator-facing ambiguity, not deliberation, and stays.
- Docs list (focus 6): not a defect.
  `start`'s `Long` text and the overview's loom entry both already document `driver_not_parked`, `internal/fabricengine/doc.go` lists the `l`-in entry points, and no `manifest/designs/` doc covers loom or fabric, so the named three satisfy the same-commit docs rule.
