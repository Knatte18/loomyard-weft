```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out (F1, F2, F4), as the fix-proposal lens.

Proposals checked: F1 (anchor only on a settled parent entry, re-wait, fallback stays homeless), F2 (renumber only the windows homed on the desktop), F4 (SIGTERM runs no `window_command`). F3 was not carried out as proposed: the parent declined the probes, so it is out of this lens.

- **MEDIUM.** The changed D9 text "A renumber gives the present windows whose home is that desktop, in their current order, the positions in key order that those windows held."
  - Summary: with the new wording, "present" can mean "open anywhere" as well as "on this desktop". An overflowed window is then present and homed on H but sits on L, so a renumber of H must place it in a current order it does not have on H.
  - Evidence: the old wording, "the desktop's present windows", tied presence to the desktop. The rewrite moves the desktop into the home clause, and D9 elsewhere uses "absent" for "a window not yet reopened", which is the "open anywhere" sense. Take H holding A (key 0) and B (key 2) on H, with overflowed C (key 1) open on L. If C counts as present, it needs a current order on H and has none; if it counts as absent, it keeps its position. The text does not say which. The new sentence "A present window homed elsewhere ... keeps its home and order key" covers the windows on L only, not C's standing on H.
  - Fix direction: say "the windows on that desktop whose home is that desktop", and count a window homed there but sitting elsewhere as absent, so it keeps its position.
- **LOW.** The changed D8 text "Its 180-second bound runs from the child's first entry into the wait and covers every return to it."
  - Summary: a child that first finds its parent's entry unsettled under the lock only after taking the lock has not yet entered the wait, so the text does not say when its bound starts.
  - Evidence: D8 sends a child into the wait only with "no parent entry, or an unsettled one". A child whose parent's entry was settled outside the lock, then went unsettled again when the parent's window reopened and its pre-wait write landed (D9), first enters the wait from under the lock. "From the child's first entry" then starts the bound late, which is fine but unstated. The new D8 acceptance criterion, "every return to the wait falls within one 180-second bound", assumes the bound started earlier.
  - Fix direction: anchor the bound to the child's start, or say that a first entry from under the lock starts it.
- **LOW.** The changed D12 text "A child ended by SIGTERM runs no `window_command`, and it ends one already running before it exits."
  - Summary: a SIGTERM that arrives during the parent wait or the lock wait now ends the child with no `window_command`, and that case is not named.
  - Evidence: the round-1 F13 fix promised `window_command` on every exit path so the operator's script still runs. The record now drops it on the SIGTERM path, but its stated reason, "a command started then could not count on finishing", holds only when the SIGTERM comes from `runproc.End`. D10 registers the child in `runproc` only "when the opening verb runs inside a run". An unregistered child, such as one from `lym machine start`, can be sent SIGTERM by something other than `runproc.End`, for example a logout, and no 5-second SIGKILL grace follows. That reason does not cover this case.

No further tool calls made.
