---
round: 4
focus:
  - "D5 in decision-record.md, revive versus a hub pause: shuttle's send accepts a landing when the input box clears or a turn start appears, and the pause reads busy only from hook-derived state. Settle whether, with every covered session idle-stalled, the Decision still lets a revived turn run under a live pause entry unrecorded, and whether the stated bound and its Acceptance criterion are achievable as written."
  - "D5 in decision-record.md, the conditioned send: revive holds the pause record's write lock across each send, and that lock blocks with no timeout. Settle whether the Decision states the longest time the lock is held per send, including when a draft, operator activity or an unmatched turn start is present on an idle-stalled api-error session."
  - "D6 in decision-record.md, cancellation during the hold: settle whether the claim that a call cancelled during the hold ends with the cancellation at release holds for a caller context whose WithTimeout deadline passes during the hold first, and whether the Acceptance criterion matches."
  - "D8 in decision-record.md, the watcher re-exec: settle whether the claim that no in-memory pause handling is lost is true of the watcher's memory-only handled set, for an entry live at the re-exec whose grace end is still ahead."
---

Each entry restates the sites it depends on; none caps severity or pre-states a verdict.
