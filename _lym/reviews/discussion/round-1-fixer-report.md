# Fixer report — discussion round 1

F2, a `decision`-class finding, went to the parent and is left out here.

## Changes

- F1 (fixed as proposed): D10 widens the removed-holder sweep's subjects (`sweepSubjects`) and run condition to strands holding a read, marked, unanswered message; a holder gone by the two-miss rule has each such message settled `unanswered` with reason `holder-gone`, and a waiting `--wait-reply` is answered. The D10 acceptance row adds the case of a holder that holds no other mail or inject.
- F3 (fixed as proposed): D10 journals the consumed chase time in a `chase-aged` record written at `agedPersistInterval` and when the clock stops, and the restart derivation reads it; the parity claim now says the clock stops when the holder is dormant or paused, as an inject TTL does, and also under another sender's hold, which an inject TTL does not. D11 names the record; the D10 acceptance row adds a restart after a pause.
- F4 (fixed as proposed): D8's rationale says shuttle send refuses under a pause but bypasses creel's journal, queue and hold; its Bound states the queue-under-pause, idle-only typing, no appended `MessageTail` and inject-authority changes.
- F5 (fixed differently): the proposed way forward, posting to the original sender by name, fails for a program sender, whose mailbox nobody reads, and an EXTERNAL one, which `checkReceiverLocked` refuses as a receiver. D10 now refuses a forward of a chase (way forward: reply to the chase, or forward the chased message), and accepts a reply to a chase only from the chased message's current holder; a former holder is refused and pointed at the current holder through `lym creel log --id`. The acceptance row matches.
- F6 (fixed as proposed): D2's Bound states when a program claim is accepted on and off Linux, that verbs such as `lym loom decision add` post caller text as a program, what bounds it, and that program trust is not narrowed to named programs.
- F7 (fixed as proposed): D3 has every hold-notice notifier wiring resolve with batten's run-notice rule, and names the burler, loom and webster `SetNotifier` wirings that change; Scope gains a line for it.
- F8 (fixed as proposed): D3's notice text and D5's orch bullet answer a pending marked question with `--reply`, and post to the strand only otherwise; a new D3, D5 acceptance row.
- F9 (fixed as proposed): D10's retention carries a read, marked, unanswered message forward; D12 names the seven-day `journalRetention`.
- F10 (fixed as proposed): the chase number and the chased message's id travel on the chase's own `posted` record; the restart also writes a missing `chased` record from it.
- F11 (fixed as proposed): a chase and the sender notice name an EXTERNAL sender by the bare name.
- F12 (fixed as proposed): D10's Bound states the pane-mail and hook-strand cover for a never-read marked message, and that a waiting sender waits until its `--timeout`, or without end when it passes none.
- F13 (fixed as proposed): D11 has a log entry carry a reply's reply-to id; a D11, D12 acceptance row checks the pairing.
- F14 (fixed as proposed): D5's Rationale cites D10.
- F15 (fixed as proposed): the cycle constraint carries the `go list -deps ./internal/creelcli` evidence, which this fixer re-ran.
- F16 (fixed as proposed): the Scope bullet is cut to its requirement; the reasoning moved into support-log.md's Rejected alternatives entry on each engine's own messaging.

## Self-check

- fix-consistency, LOW, D2 Bound's two program-claim sentences disagree: fixed as proposed (merged with the fix-claims finding).
- fix-consistency, LOW, the `PATTERN-refusal-way-forward` constraint omits D10's refusal: fixed as proposed; it names both D10 refusals.
- fix-consistency, LOW, Scope lacks the notifier-wiring change: fixed as proposed.
- fix-consistency, LOW, no acceptance row for the reply-to id: fixed as proposed (the D11, D12 row).
- fix-consistency, LOW, `PATTERN-daemon-wakeups` constraint omits the widened sweep: fixed as proposed, naming its existing backoff.
- fix-consistency, LOW, support-log ledger still says the clock is frozen like an inject TTL: fixed as proposed.
- fix-claims, LOW, the off-Linux program claim is accepted whenever the strand name names no strand of this hub, not only when unset: fixed as proposed.
- fix-way-forward, MEDIUM, and fix-proposal, MEDIUM (merged), the way forward for a non-holder's chase reply fails for program and EXTERNAL senders: fixed as proposed, taking the forks' option that chases cannot be forwarded, with a former holder pointed at the current holder, as F5 above records.
- fix-way-forward, LOW, "bounded only by its own call timeout" hides that the default wait never ends: fixed as proposed.
- fix-proposal, LOW, a crash leaves no `chased` record for D11's log: fixed as proposed; the restart writes it.
- fix-proposal, LOW, the D3 wiring sentence also covers loom's non-notifier calls: fixed as proposed; it names the runners' `SetNotifier` wirings and leaves the hand-back and landing notices.

## Disputed

None.

## Deferred

None.
