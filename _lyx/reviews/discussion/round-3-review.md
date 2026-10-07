---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:109
    summary: D7 lock-held refusal names lyx loom status as the place the lock holder shows, but status shows no lock holder for a halted run
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:112
    summary: D7 no-live-driver refusal sends a run with no driver strand in reed state to lyx batten run, which D6 makes a no-op for it
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:88
    summary: batten halted-wait reason and awaiting hint still name lyx loom start as the resume command, and D6 halt notice is not tied to the text that carries it
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:173
    summary: Acceptance lock-held row requires no live driver, while D7 refuses a halted run with a held run lock whatever its driver
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:58
    summary: D3 calls the 60-minute deadline a constant a test can shorten while Constraints call it a parameter, and no Decision names who passes it
---

### [MEDIUM] D7's lock-held way forward points at a status display that does not exist

**Location:** `_lyx/discussion/decision-record.md:109` (D7 outcome list), repeated in Constraints (line 136) and Acceptance criteria (line 173).
**Issue:** The halted-with-held-run-lock refusal names "`lyx loom resume` again once `lyx loom status` shows the lock holder gone", and the Constraints and Acceptance rows repeat "a wait for a lock holder shown by `lyx loom status`" and "refusal naming `lyx loom status`".
`lyx loom status` shows no lock holder.
`internal/shedverbs/status.go` builds a fixed core of keys, and loom registers no `StatusExtras` hook.
The `waiting` note is read only while the state is running (`waitingNote` returns empty otherwise), and the run in this outcome is halted.
So the refusal's way forward cannot be followed as written, which `PATTERN-refusal-way-forward` forbids; a way forward must be a real step.
**Fix:** Name the way forward the way start's own not-parked refusal does: the driver is still finishing, retry `lyx loom resume` in a few seconds.
Drop `lyx loom status` from that bullet, from the matching Constraints clause and from the Acceptance row.
Adding a lock-holder field to status would widen the task, so it is not the fix.

### [MEDIUM] D7's no-live-driver refusal names a verb that does nothing for a run without a driver strand

**Location:** `_lyx/discussion/decision-record.md:112` (D7, last outcome), with D6 at line 87.
**Issue:** For a halted run with no live driver strand, the refusal names `lyx batten run <slug>` first, "which brings the driver back (D6)", and `lyx loom start --no-attach` only "for a run batten does not hold or once D6's revive has failed".
D6 revives only when reed state holds a driver strand that is not live; with no driver strand it does nothing, "no revive, no warn".
A Go-driven halted child has no driver strand, and neither does an LLM-driven one whose strand left reed state.
For both, the refusal's lead step is a provable no-op, and neither stated condition for the `start` alternative matches them.
`lyx loom resume` cannot read the seed's driver, but it already reads strand liveness and can tell a present dead driver strand from an absent one without it (`PATTERN-driver-choice-single-site` holds).
**Fix:** Split the way forward on the strand table.
A dead driver strand present: `lyx batten run <slug>` from the prime, then `lyx loom resume` again, or `lyx loom start --no-attach` once D6's revive has failed.
No driver strand in state: `lyx loom start --no-attach` in the task worktree.
Add the two cases to the Acceptance row for the no-live-driver refusal.

### [MEDIUM] The standing halt reason still tells the orch to run lyx loom start

**Location:** `_lyx/discussion/decision-record.md:88` (D6 third bullet) and `:102`/`:121` (D7, D8); code at `internal/battenshed/innerrun.go:49` (`haltedWaitReason`) and `:391`, and `internal/battenshed/doc.go:21`.
**Issue:** D7 exists so the orch stops calling `lyx loom start` for a halted child.
The text the orch reads through `lyx batten status` for a halted child is `haltedWaitReason`, which says `run "lyx loom start" in the task worktree to resume it`; the awaiting hint at line 391 says the same.
No Decision changes either, and D8 lists only the orch stencil and `doc.go` updates.
D6 says "the halt notice names the way forward (`lyx loom start --no-attach`)", but the notice batten sends (`renderNotice`) carries no way forward, only the attach command; the text that does carry one is the stuck reason.
The record also lists notice wording as Out, so it is unclear whether D6 touches the notice or the reason.
**Fix:** In D6, name the stuck reason as the carrier: for a halted child it names `lyx loom resume` in the task worktree while the driver is live or was revived, and `lyx loom start --no-attach` after a failed revive.
State that the notice line and the notice channel are unchanged.
Add `haltedWaitReason`'s text and the batten `doc.go` sentence to D8's docs, and an Acceptance row for the reason.

### [LOW] Acceptance lock-held row is narrower than D7

**Location:** `_lyx/discussion/decision-record.md:173`.
**Issue:** D7 refuses any halted run whose run lock is held, ahead of the driver bullets.
The Acceptance row reads "halted with the run lock held and no live driver", so a held lock with a parked live driver is specified by D7 and covered by no row.
**Fix:** Drop "and no live driver" from the row so it matches D7's order (and F1's retry wording).

### [LOW] The verify deadline is called a constant and a parameter

**Location:** `_lyx/discussion/decision-record.md:58`, Constraints line 142.
**Issue:** D3 says "a named constant, 60 minutes, that a test can shorten"; a constant cannot be shortened, and Constraints say "the verify deadline is a parameter a test shortens with the 60-minute constant as the production value".
Neither says who supplies the value to `verifytree.Verify`, so the testability requirement has no carrier.
**Fix:** In D3, state the deadline as a value the verify run is given, with production callers passing the 60-minute constant and a test passing a short one; keep the constant itself.

## Focus verdicts

- Wait mark failure handling (D2): satisfied.
  D2 states that any mark-op failure, at a shuttle site or in the landing callback, is logged and changes no verdict, error return or gate outcome, and the Acceptance row on the landing callback covers both callers.
  Not a defect.
- Pair-wide `reed resume` revive (D6): the Bound names what else the resume relaunches (every non-live, non-hidden strand, each at its prompt with nothing typed) and what limits the revive (once per halt episode per batten process, no shed step, no marker removal, no resume line).
  It tells a Go-driven child from a dead LLM driver by the absence of a driver strand in reed state, without the seed.
  I confirmed in `internal/loomcli/bootstrap.go` that the driver strand exists only for an LLM-driven run, and in `planResumeLaunches` that the relaunch set is as stated.
  The residual gap is the dead end F2 names.
- Goal, Scope and D5: they promise only what D5 delivers; the finished gated strand's relaunch is stated with its bound (no prompt typed, retired at the next teardown), and a pre-existing output file cannot fake a finished ungated run because `Spec.validate` rejects one.
  Not a defect.
- Verify deadline (D3): the process-group setup is placed in `verifyrun`, whose stated import limit is the standard library and `internal/logger`; per-OS files using `syscall` stay inside it.
  The testability gap is F5.
- `lyx loom resume` (D7): every run state has an outcome, and the bootstrap-lock timing and the merge-versus-not-parked cases are settled and match `lyx loom start`'s behavior in `internal/loomcli/start.go`.
  The gaps are F1, F2 and F4.
- Open risk on tmux elapsed arithmetic: on tmux 3.6 here, `#{e|-:%s,#{@lyx_wait_start}}` expands to the elapsed seconds in a time-expanded format, so the elapsed form works on this version.
  The risk stays open for older versions, so it is not a finding.
