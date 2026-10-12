---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md (Scope In, D1, D3, D4)
    summary: this run and the in-flight run reed-multi-window both plan changes to the Go packages internal/shuttleengine and internal/shuttleengine/claudeengine
    fix: the parent decides the landing order of pause-esc-rewind and reed-multi-window, or confirms the two may proceed in parallel and the later one rebases over the earlier in these two packages
---

## Scope check

The design matches the board entry `pause-esc-rewind`, with no scope deviation.

- The entry's three checks are each answered and acted on.
  More than one Esc, and an Esc into an idle prompt with a background shell: D1, D3 and D4 send Esc only into a pane showing a running turn, and any second Esc only after a 2-second settle and a fresh pane read.
  Resume and the picker: D6 keeps the guarded send, which already refuses a pane with no input box, and names the refusal by the guard's own reason; Rejected alternatives explains why resume does not close or detect a picker.
  `interrupted` against `still_busy`: D5 redefines the report lists and adds `no_running_turn` and `background`.
- The entry's risk, resume typing into an open picker, is covered by D6 and a criterion for a pre-change entry.
- The "also seen" check on shed steps running under the pause is answered as intended by the shipped design (Scope Out, Q5), and D2's `background` list makes them visible.
- The additions beyond the pause engine follow from the same defect: the orch watcher repeats the two-Esc loop (D4), and the guard sits in `Runner.Interrupt` so `lyx shuttle interrupt` and the planned `creel` interrupt inherit it (D3, Scope In).
  The new PATTERN entry (D7) is the repo's rule for a new cross-cutting invariant.
- The design contradicts and duplicates no board entry.
  `creel` plans to move `lyx shuttle interrupt` behind creel and asks shuttle how keys reach a pane safely; D3 puts the guard in shuttle and states the `ErrNoRunningTurn` contract creel relies on.
  `run-window-bugs` reworks `claudeengine`'s `Startup` and shuttle's readiness probe; the design leaves `Startup` alone (plan note), and its `judgeReadiness` change is a predicate extraction, not the probe.
  `orch-split` and `creel` will later move the orch watcher's delivery; the guard in shuttle is what they inherit (plan note).

## Overlap with runs in flight

`lyx loom peers` lists two peers and no parent commits.

- `cleanup-1012` (PR-Align) changes `internal/fabricengine`, `internal/lyxcwd` and `tools/tokencount`; no overlap.
- `reed-multi-window` (Discussion-Burler-Review, no plan, no changed paths yet) plans changes to `internal/shuttleengine` (`spec.go` and `attach.go` for the anchor default, a new `lyx shuttle open` verb) and to `internal/shuttleengine/claudeengine` (D12's launch line, session name flag and resume of a session with no turns).
  This run plans changes to the same two packages: `Runner.Interrupt`, `Run.Interrupt`, the shared running-turn predicate in `readiness.go` and `sessionstate.go`, and a new running-turn pane reading and settle time in `claudeengine`.
  The files and concerns named in the two records differ, so a text conflict is unlikely, but both runs change the same two Go packages, which is the overlap rule's escalation case (F1).
  Neither record changes a PATTERN entry the other changes; this run adds a new PATTERN entry and repoints `PATTERN-pane-text-injection`'s background, and `reed-multi-window` changes no PATTERN entry.

### [MEDIUM] Shared Go packages with reed-multi-window

**Location:** `_lyx/discussion/decision-record.md` (Scope In, D1, D3, D4)
**Issue:** `pause-esc-rewind` and the in-flight run `reed-multi-window` both plan changes to `internal/shuttleengine` and `internal/shuttleengine/claudeengine`; the board and the records set no landing order between them, so the parent decides it.
