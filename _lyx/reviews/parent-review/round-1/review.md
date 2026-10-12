---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/discussion/decision-record.md (D8, Constraints docs list)
    summary: D8 retires the bare batten and loom pause, but the orch role stencil still tells the orch to hold a run with it, and the record names no replacement for that mid-row hold.
    fix: In D8, name the replacement for the orch's hold-then-goto use of `lyx batten pause <slug>` (`contracts/stencils/orch/orch-template-role.md` line 30 and the loom pause after Plan-Write at line 90, mirrored in `.lyx/orch/role.md`), either `lyx pause <slug>` or `pause --after <current_producer>`, stating whether it still stops a run mid-row so a goto can move it; add those stencils, the shed driver stencil and the `contracts/stencils/claims_test.go` claim on batten pause to the Constraints docs list as same-commit edits.
  - id: F2
    severity: BLOCKING
    class: scope
    location: _lyx/discussion/decision-record.md (D4)
    summary: The goal says a pause holds new agent starts, but D4 holds only shuttle run starts, so reed revive, batten's driver revive and `lyx loom start` can still spawn agents under a pause.
    fix: In D4, list every path that spawns an agent session (shuttle run start, `lyx reed resume` and the reed watchdog revive, batten's driver-dead revive, `lyx loom start` and `lyx batten run`) and state for each whether it waits under a live entry or why it need not; add an acceptance criterion for each that waits.
  - id: F3
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md (D3, D6)
    summary: The writer's own lens forks reported races the record does not settle (a resume during a pause's grace, overlapping hub and slug entries, a pause killed between the Esc and the record write), and their findings were never folded in.
    fix: Fold the lens findings into the record, settling in D3 and D6 what a resume during the grace does, how a hub entry and a slug entry for the same run interact on pause and on resume, and how a session interrupted before the entry recorded it is resumed; add the medium lens findings (shuttle deadline under an unplanned sleep, record placement and leaf-package fit, refusal-spec rows, creel and runs-overview fit) or reject each with a reason in the support log.
---

### [BLOCKING] D8 orphans the orch's hold-then-goto step

**Location:** `_lyx/discussion/decision-record.md` D8 and Constraints; `contracts/stencils/orch/orch-template-role.md:30` and `:90`.
**Issue:** The board entry wants the step-boundary pause gone, which D8 does, but the orch stencil's step 3 holds a run with `lyx batten pause <slug>` before `lyx batten goto`, and its parallel-run guidance pauses with `lyx loom pause`. The record neither names what replaces that use nor lists those stencils among the same-commit edits, so the change leaves the orch with a refused instruction.

### [BLOCKING] D4 does not hold every new agent start

**Location:** `_lyx/discussion/decision-record.md` D4.
**Issue:** The goal and the board's proposal say a pause stops every lyx process from starting agent work, yet only the shuttle run start waits. Reed's revive, batten's driver revive and `lyx loom start` start sessions outside that entry point, so a suspend can still catch a fresh agent turn.

### [BLOCKING] The lens findings are not folded into the record

**Location:** `_lyx/discussion/decision-record.md` D3, D6; `_lyx/discussion/support-log.md`.
**Issue:** The writer reported its lens forks returned after the gate passed, so the record under review does not reflect them. The ordering and overlap questions they raise decide whether a pause is safe, which is the board entry's goal.
