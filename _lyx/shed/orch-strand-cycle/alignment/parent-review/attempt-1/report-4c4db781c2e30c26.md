---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: decision
    location: _lyx/discussion/decision-record.md, D1 (Departures from the operator decision of 2026-10-21)
    summary: D1 departs on its own from the liveness rule the operator decided on 2026-10-21 and wrote into the board entry, in an autonomous run with no operator answer
    fix: The operator rules on each of D1's three departures (session instead of session and window, not-the-shell instead of is-the-engine, condition 3 skipped on Windows); the decision record then states the ruling in D1, drops the Departures list for any departure the operator overrules, and moves each accepted one out of the auto-mode assumptions into an operator decision.
---

### [BLOCKING] D1 overrides the entry's operator decision without the operator

**Location:** `_lyx/discussion/decision-record.md`, D1.

**Issue:** The board entry's section 1 records an operator decision (2026-10-21): a strand is live only when its pane exists and is not dead, sits in the reed session and window it was launched into, and runs the engine (`claude`) in the foreground, not a shell.
D1 keeps the first condition and changes the other two, and adds a third change the entry does not make:

- Condition 2 checks the reed session only, so a pane moved to another window of the session reads live.
- Condition 3 tests that the foreground command is not reed's launch shell, so a pane where another program runs after the engine exited reads live.
- Condition 3 is skipped on Windows, so on Windows a strand whose engine exited reads as it does today.

The record names these as departures and gives reasons: `PATTERN-shuttle-provider-seam` keeps engine names out of reed, a window-exact check would make `start` launch a second orch beside a moved live one, and a PowerShell cmdlet runs inside the shell.
The reasons are plausible, but the run is autonomous: the support log records every pick as an auto-pick with no operator answer, so no one with authority over the entry accepted the departures.
This is not a deviation the writer can fix alone either.
Conforming to the entry's text on condition 3 conflicts with a PATTERN invariant the entry does not address, and conforming on condition 2 brings the duplicate-orch hazard the record names.
The board and the records do not settle which wins, so the question goes to the operator.

The rest of the design fits the entry:
sections 2 to 4, the folded `reclaim-dead-strand-names` (#799, #802 and the name-clash part of #811) and the folded #828 each map to a decision (D3 to D7), and the out-of-scope items match the entry (the `loop-exited` halt goes to `loop-exit-halt-names-its-cause`, the hub rules section is removed by the orch after landing).
D3's narrowing of the `status` restart to a live strand follows from the watcher needing a strand, and `start` relaunches both together.

## Board entries that touch this task

- `batten-cancel-verb` depends on this task; D1 states its consequences for Resume and shuttle stop, which that entry builds on. No conflict.
- `loop-exit-halt-names-its-cause` owns the `loop-exited` halt of #811; the record leaves it out. No duplication.
- `creel-agents` (reply discipline): D4 makes the note request expect no answer but `defer`, which agrees with it.
- `spawn-audit-log`: D3 adds a spawn site on the existing watcher path; the note's recorded launch path is unbuilt. No conflict.
- `codex-engine`: a second engine is planned, which supports keeping reed engine-neutral; it does not decide F1.

## Peer runs and parent commits

`lyx loom peers` lists no parent commits and four peers, none of which overlaps this run's area (reed, orch, creel, shuttle, seatengine):

- `loop-exit-halt-names-its-cause`: `internal/loomcli`, `internal/loomengine` anomaly, `internal/shedverbs` loop. No overlap.
- `retry-archive-tag-push`: `internal/fabricengine`, `internal/gitrepo`. No overlap.
- `done-check-spaced-create-paths`: `internal/planglyph`, `internal/planparser`, `internal/websterengine` record batch. D1 changes what webster's `StrandLive` reads through reed `Status`, not webster's record batch. No overlap.
- `fork-git-guard-read-only-tag`: `internal/burlerengine` cluster. No overlap.

No landing order applies; every peer's `row` is 0 (unknown order) against `own_row` 3.
