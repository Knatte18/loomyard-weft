---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:43
    summary: Decision 3's wait bound covers only status-push waiters; the Open risks entry says a hung holder runs until git gives up, but gitexec sets no command timeout, so a hung status push now holds the lock and blocks detached children with no bound
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:31
    summary: A single failed side returns its error as it is, a bare sentinel naming no side, so the caller and the commit-records envelope cannot tell which side failed; only the both-failed case names sides, and the envelope text for a code-side-only failure is left unstated
  - id: F3
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:43
    summary: Decision 3 bounds only the status-side wait, but the new entry also serves lyx fabric push normal mode, which after a landing commit contends with its own just-spawned detached child; whether push waits bounded or blocking is not decided
  - id: F4
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:68
    summary: The refusal-way-forward constraint demands a way forward correct for a real divergence, but no Decision names it, and under Decision 5 both lyx fabric push and a commit-records re-run error again on a divergence
  - id: F5
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:21
    summary: Scope Out and Decision 4 refer to a shared primitive and the primitive that no Decision introduces, while Decisions 1 and 3 name only the new entry and PushAnchored
---

## Basis

Judged against the rubric and the focus directive, with the claims checked in `internal/lock/lock.go`, `internal/gitexec/gitexec.go`, `internal/gitrepo/push.go`, `internal/fabricengine/{pushanchored,coalesce,spawn,commit,weftgit}.go`, `internal/fabriccli/weft_verbs.go`, `internal/loomcli/{commitrecords,wiring}.go`, `internal/statuscommit/statuscommit.go` and `contracts/specs/refusal-spec.md`.
The writer stencil tells the writer not to gather exact signatures, `file:line` citations, interface shapes or dependency lists; none of the findings asks for one.
No relocation finding: the deliberation that sits in Decisions is rationale or bound text, and the rejected alternatives are already in `support-log.md`.
Focus question 4 (the PATTERN entry's own text) holds: Decision 6's entry text carries the prime's records-only per-transition exemption, and Constraints name the `commit-records` row; F4 covers what that row must say.

### [MEDIUM] Decision 3's bound stops at the waiters, and "until git gives up" is not a bound

**Location:** `decision-record.md` Decision 3 (line 43) and Open risks (line 83); `support-log.md` Rejected alternatives (line 28) and Question ledger Q7.
**Issue:** `lock.AcquireWriteLock` blocks with no timeout and `gitexec` sets no command timeout (`exec.Command`, no context), so the bounded wait Decision 3 adds bounds only a status push that is waiting for the lock.
Two edges are left unbounded and unstated.
First, the status push that does get the lock runs `git push`, a fetch and a retry with no limit, so a stalled network still stalls the transition itself, as it does today; the support log's rejection of a no-timeout wait argues "a stalled network would stall the run's own transition", which the chosen design does not prevent.
Second, that push now holds `fabric.push.lock`, which it did not before, so a hung status push blocks every detached child on its blocking `CoalescePushBothAt` acquire, and each commit that lands spawns another blocked child.
The Open risks entry covers only a hung child and says it holds the lock "until git gives up", which is no bound when no timeout exists.
**Fix:** In Decision 3, state that the wait bound covers lock waiters only, that the holder's git calls are unbounded as today, and that a hung status push therefore blocks detached children behind it until git returns, accepted.
Correct the Open risks entry to say the same for a hung holder of either kind, and narrow the support log's rejection rationale to what the bound actually buys.

### [MEDIUM] A single failed side names no side

**Location:** `decision-record.md` Decision 1 (line 31), Decision 2 (line 38), Acceptance criteria (lines 89-90).
**Issue:** Decision 1 returns a single failed side's error "as it is", a rejection as the bare `gitrepo.ErrPushRejected`, and only the both-failed `errors.Join` carries a message naming the side.
In the common single-failure case the error text names no side, so a caller or an operator reading the seam warning or the `commit-records` envelope cannot tell which side failed.
Decision 2 says the envelope "no longer claims the records were not pushed" but not what it claims, and for a code-side-only failure the records side was pushed, so the current "the commit landed locally but was not pushed" wording is false.
The bare form only matters to `PushAnchored`, whose unwrapped sentinel is pinned for batten's records-only caller; `errors.Is` is enough for the new entry's callers.
**Fix:** In Decision 1, make every failure from the new entry name its side and wrap with `%w`, so `errors.Is(err, gitrepo.ErrPushRejected)` holds for a single side and for the join alike; leave `PushAnchored`'s unwrapped sentinel as it is.
In Decision 2, state that the envelope and the seam warning carry the side-named text and that the `commit-records` message does not claim the records were unpushed when only the code side failed.
Add that single-side case to the acceptance criterion that now names only the mixed-kind case.

### [MEDIUM] `lyx fabric push` inherits a status-side wait bound it was not given

**Location:** `decision-record.md` Decision 3 (line 43) and Decision 5 (line 52).
**Issue:** Decision 3 bounds "the status-side acquisition, in `PushAnchored` and in the new entry", yet Decision 5 routes `lyx fabric push` normal mode, an operator command, through the same entry.
That verb first runs `Fabric.Commit`, which spawns a detached push child whenever it lands something, and then pushes in-process; the in-process push now contends with its own child on the absorbing lock.
Whether it waits without limit, as `PushCoalesced`'s repo-root lock did, or for the bounded time and then returns an error envelope on a healthy run is not decided.
**Fix:** Decide in Decision 5 whether the wait in `push` normal mode is the bounded one, and state the outcome on expiry; if it is, say that the bound is long enough or that expiry is an error envelope with the way forward `commit-records` names, and add a criterion that `push` against a held lock behaves as stated.

### [MEDIUM] The `commit-records` way forward is demanded but never chosen

**Location:** `decision-record.md` Constraints (line 68), Decision 5 (lines 52-53).
**Issue:** The constraint says the `commit-records` push-failure message and its refusal-spec row must name a way forward "correct for a real divergence as well as for a transient failure".
The row today names `lyx fabric push` or a re-run of `commit-records`.
Under Decision 5, `lyx fabric push` against a real divergence returns an error envelope and rewrites nothing, and a re-run of `commit-records` pushes the same way, so both ways forward fail again on a divergence.
No Decision says what the operator is to do, so the constraint states a requirement the record does not satisfy.
**Fix:** Add to Decision 2 the way forward for a real divergence, in the shape the Publish push-rejected row already uses (the operator merges the remote branch into the local one in the named worktree, then re-runs), with no command `PATTERN-no-denied-recovery` forbids; keep the transient way forward alongside it.

### [NIT] "The shared primitive" is never introduced

**Location:** `decision-record.md` Scope Out (line 21) and Decision 4 (lines 47, 49).
**Issue:** Scope Out says batten "gains the lock and the retry through the shared primitive", and Decision 4 starts "the primitive fetches"; Decisions 1 and 3 name only the new entry and `PushAnchored`, and Decision 4 itself says the retry lives in both of those.
A reader cannot tell whether a third shared unit is intended.
**Fix:** Replace both phrases with the named units, or add one sentence to Decision 1 saying the two entries share one per-side step.

## Focus departures

None.
