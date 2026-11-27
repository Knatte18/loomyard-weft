---
round: 3
focus:
  - "D8 and D9 of decision-record.md, with D1's modes: where does a child anchor when its parent's places entry is unsettled, when the parent is under mode off, when the parent's full home overflows it, when the parent has no home, and when the parent wait runs out? Settle whether lock order can change the anchor, what each case records as the child's home, and whether the wait is bounded and releases the lock."
  - "D9 of decision-record.md, renumber and order keys: after a window overflows from its full home onto another desktop, can a later placement or renumber on that desktop change the overflowed window's home or order key, or leave two entries with one key on a desktop?"
  - "The Constraints section of decision-record.md against D3, D4 and D6: is every KWin and kscreen-doctor behaviour the design rests on recorded with its observed result and the command or source that pins it, and is any fact still left for Plan or an implementer to find out? For a fact the parent declined to have observed, check that Open risks names the decision resting on it."
  - "D12 with D9 and D10 of decision-record.md: on a SIGTERM exit, does window_command run, given that runproc.End sends SIGKILL after endGrace (internal/runproc/end.go) and desktophook bounds window_command in internal/desktophook/window.go productionTiming?"
  - "D3, D4 and D6 of decision-record.md: does the query report what the apply and the zone choice need, including a window identity that separates two windows sharing one pid and the zone boxes and padding for an untiled window, and is the KWin script plugin name unique when queries run at once outside the lock?"
  - "D9 and D10 of decision-record.md, places file and process bounds: what happens to writes and to the parent wait when the places file is undecodable (state.UpdateJSON), what a window appearing after the window wait records as its last and home desktop, and whether the hidden verb's stated bound covers all it reads and runs."
---

The first entries follow the open ledger keys.
The last entries name places in the record where the question is not yet settled by a raised finding.
