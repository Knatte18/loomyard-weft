---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
---

# Round 2 fixer-report

## Changes

- F1 (D8, D9, D10, acceptance): the anchor is read only from a settled parent entry.
  A child that finds the parent's entry missing or unsettled under the lock releases the lock and returns to the parent wait, and one 180-second bound covers every return.
  The sentence that gave an unsettled entry's home or last desktop as the anchor is gone, and the rationale names the three cases where that desktop is wrong.
  A window placed from the current-desktop fallback takes no home (D8, D9 and both D8 bounds), so its next placement anchors on its parent again.
  D10's lock step names the return, and the second D8 criterion is replaced as proposed and split into two criteria.
- F2 (D9, acceptance): a renumber covers only the windows that sit on the desktop and are homed on it, and the trigger bullet compares only those.
  A window sitting there but homed elsewhere keeps its home and key, and the fourth D9 criterion is extended as proposed.
- F3 (Constraints, Open risks): the facts were put to the parent (creel #2818) with the read-only and window-moving probes.
  The parent declined every probe (#2819), so, as the fix's own fallback says, the facts moved to Open risks, each naming the decision it rests on, and the "before code relies on it" rule is gone.
  Following the parent's reply, Open risks names the probes as the implementing run's first step: read-only ones it may run itself, the moving ones only with the operator.
  The parent's `kscreen-doctor -o` and `kwinrc` readings were recorded in Constraints.
- F4 (D12, acceptance, support log Q18): a child ended by SIGTERM runs no `window_command` and ends one already running, and the first D12 criterion covers the case.
- Support log: a Round 2 section, Q18 updated and Q20 added for the homeless fallback.

## Self-check

- fix-consistency, LOW, D1 still pointed at Constraints for the probe: fixed; it now points at Open risks.
- fix-consistency, LOW, the second D8 bound missed the homeless fallback: fixed.
- fix-consistency, LOW, the Review help-text criterion missed the SIGTERM exception: fixed.
- fix-claims, LOW, D12's SIGKILL reason covers only a child that `runproc` registered; merged with fix-proposal LOW 3 and fix-way-forward LOW 3: fixed; the reason now says a SIGTERM means the run or session is ending, with `runproc.End`'s SIGKILL as one source.
- fix-proposal, MEDIUM, a renumber's "present windows" was ambiguous for a window homed there but sitting elsewhere: fixed; the renumber covers the windows that sit on the desktop and are homed on it, and any other entry homed there counts as absent and keeps its position; the D9 criterion was extended to cover it.
- fix-proposal, LOW, the bound's start was unstated for a first entry into the wait from under the lock: fixed; the bound runs from the first entry, whether before the lock or from under it.
- fix-proposal, LOW, a SIGTERM during the waits drops `window_command` without being named: fixed; D12 says "at any step".
- fix-way-forward, MEDIUM, the read-only probes need the user session bus: fixed; the run probes itself only where its session reaches that bus, and otherwise asks its parent to have the operator run them.
- fix-way-forward, MEDIUM, a contradicting probe named no actor: fixed; the run stops and sends the probe's command and output to its parent, which reopens the decisions.
- fix-way-forward, LOW, ending a running command needs a cancel path: fixed; D12 says the window-command runner gains a way for its caller to cancel a command and its process group.
- fix-way-forward, LOW, the child's total wait was unbounded across returns: fixed; the 120-second lock timer is summed over one child's acquires (D10, with a D10 criterion).

## Disputed

None.

## Deferred

None.

## Rulings

None.
