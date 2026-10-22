---
verdict: APPROVED
---

## Scope against the board entry

The entry `tender-pause-holds-reprompt` (issue #762) has no separate Done-when; its goal is its four Expected points, and the design covers each one:

- No new prompt reaches a paused scope from a gate, a driver or any other lyx program, and a re-prompt waits for the resume: D1 refuses at shuttle's one typing chokepoint by the target's own slug, D2 makes every caller of that chokepoint hold or visibly refuse, and D3 holds the gate loop's boundary while covered.
- A pause stops or holds a running verify gate command in its scope: D4 stops the command's process group after the covering entry's grace end and reruns it after the resume, and D5 wires every production verify site.
- `lyx creel interrupt` of a covered session interrupts it: D7, with D6's claim helper so the resume still gives the session its line.
- `lyx webster begin-batch` refuses under a covering pause and tells Master to end its turn: D8.

D8's hold of `recover-batch`, D2's claim of a refused tender resume line, the seat notice retry and the creel delivery mapping go past the entry's letter but follow from its "no new prompt from any other lyx program" and "no new one starts until the resume"; the support log records why, and none touches another entry's work.

## Fit with other board entries

- `run-process-ownership` depends on this task and owns driver chains, parked pushes, background shells, Monitors and webster forks under a pause; the decision record's Scope Out leaves exactly those to it, and D8 settles only the no-new-batch half of its `pause-background-work` note.
- `batten-cancel-verb` ends a run's processes; the plan-writer note says a held Verify ends with its process or context, so nothing contradicts it.
- `verify-watchdog` owns the Windows tree-kill gap; the decision record leaves it there as an open risk.
- `creel-agents` is an unbuilt note; nothing in this design contradicts it.

No finding.

## Overlap with other runs

`lyx loom peers` lists no parent commits; `own_row` is 3.

- `machine-handover` (row 9): it adds owner-labelled pause entries through a new tender seam and re-checks owners when a pause hold ends, and relies on the parent-lock exemption from slug entries that D5 also relies on.
  Both read the pause record, but neither changes the other's behaviour: this run does not change what `lyx tender pause` interrupts or which entries exist, and D6's claim records a session into whichever live entry covers a slug, an owner pause included, which is the same claim tender's resume already honours.
  Its record names this task as owner of the pause's verify re-prompt leak, consistent with this design.
  No design conflict; `machine-handover` is further along and lands first, and this run takes its changes at Merge-In.
- `quarry-csharp` (row 10): touches webster's batch fingerprint check before `begin-batch`, a different place from D8's pause refusal; text-only overlap at most.
  `quarry-csharp` lands first.
- `lenses-language-sets` (row 10): no overlap.
