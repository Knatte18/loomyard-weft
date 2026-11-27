# Card 23 — pattern-docs

**Prosa:**
- `PATTERN.md`
- `pattern/PATTERN-pair-remote.md`
- `pattern/PATTERN-run-fsm.md`
- `pattern/PATTERN-landing.md`
- `pattern/PATTERN-daemon-wakeups.md`
- `pattern/PATTERN-owner-machine.md`

**Intent:** The declared `PATTERN-pair-remote` change and the backgrounds the pusher, the landing and the waits now falsify.

`PATTERN.md`'s `PATTERN-pair-remote` line becomes, verbatim:

```markdown
- `PATTERN-pair-remote` — Detached pushes of lym's branches: one background pusher per target, records before code, owed pushes retried quietly; a push whose result its caller reads stays synchronous; a pair's remote steps hold the slug's teardown lock; teardown only via `internal/pairteardown`. (test) — [background](pattern/PATTERN-pair-remote.md)
```

`pattern/PATTERN-pair-remote.md`: the opening line takes the new rule;
"Push both sides" names the four target kinds, the board's and the landing target, every detached spawn starting the pusher, the interval in force and its ledger, the records hold of a standing landing entry, the synchronous pushes D1 keeps with their reasons, and the hand-commit guard;
"Pair teardown" replaces "Worktree-Teardown pushes the pair to completion" with the hand-off to the dobby and the refusal while a landing entry stands.
`pattern/PATTERN-run-fsm.md`: the landing's board `done` leaves the synchronous pushes and is the parent target's; the owed retry is once per interval; the past-the-merge read takes the local parent while origin's lacks the work.
`pattern/PATTERN-landing.md`: the local merge stays in Finalize under the parent lock, and only its push goes to the parent target's pusher; a GitHub landing waits while its parent owes a landing push.
`pattern/PATTERN-daemon-wakeups.md`: the pusher's owed retry line becomes the interval wait in `internal/runpusher/run.go`, from `fabric.push_interval_s` doubling to `fabric.push_ceiling_s`, woken by an `internal/fswatch` event on a push-now mark;
a new line lists the Go driver's network wait in `internal/sheddriver/drive.go`, from 15 s doubling to `fabric.push_ceiling_s`.
`pattern/PATTERN-owner-machine.md`: the parent target's pusher reads the parent's owner and each carried run's owner before each parent push and each run's again before its `done` write, the landing target from a Bolt it pulls first; an unreachable owner read is an owed push.
