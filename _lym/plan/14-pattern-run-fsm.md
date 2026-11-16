# Card 14 — pattern-run-fsm

**Create:**
- `pattern/PATTERN-run-fsm.md`

**Prosa:**
- `PATTERN.md`
- `pattern/PATTERN-pair-remote.md`
- `pattern/PATTERN-daemon-wakeups.md`

**Intent:**
The declared PATTERN changes.

In `PATTERN.md`, the `PATTERN-pair-remote` line under "Fabric and git" becomes:

```markdown
- `PATTERN-pair-remote` — Pushing run records or tearing down a pair: the code branch goes along after the records side, both before the run acts; a pair's remote steps hold the slug's teardown lock; teardown only via `internal/pairteardown`. (test) — [background](pattern/PATTERN-pair-remote.md)
```

A new line under "Shed and loom", after `PATTERN-shed-driver`:

```markdown
- `PATTERN-run-fsm` — Reading or acting on a run's lifecycle state: only `internal/runfsm`'s states and act rule; a transition commits and pushes before the run acts; a run whose record holds another machine's takeover hands over, never replays. — [background](pattern/PATTERN-run-fsm.md)
```

`pattern/PATTERN-pair-remote.md`'s "Push both sides" section is rewritten:
the records side goes first and the code side only once it landed, none after a failed or rejected records side;
a transition's push is synchronous through `statuscommit`, and its push owed on an unreachable origin or a held teardown lock retries behind the gate;
the detached pushes that stay (the anchor change, fabric's commit spawn, `lym fabric sync` and `add`, a lift's deferred push) keep that order in `coalescePushSides`;
the pending-flag respawn and the failed-remove respawn are gone, and a left flag waits for its worktree's next push.
The line that only `commit-records` pushes synchronously goes.

`pattern/PATTERN-run-fsm.md` is the new background:
the states and act rule live in `internal/runfsm`, and `pairteardown.ReadRun` is their one input reader;
a transition takes the gate admission, commits naming its code tip through the `Warp-SHA` trailer, and pushes records then code before the next producer starts;
an owed push holds the run at its transition, and `ReadRun` reads it from the failed-push records naming the run into `PushOwed`, whose refusal names `lym fabric push` in the owed worktree;
a rejected records push reads the run's record at origin's fetched tip, and another machine's takeover entry hands the run over through `pairteardown.HandOver`, never replaying;
a hand-over mark under the hub's `.lym` stops a second hand-over;
a code tip missing on origin waits, and the owner gives it up after `runfsm.CodeTipGrace`.
It names the tests that hold each rule (cards 5 to 8 and 12).

`pattern/PATTERN-daemon-wakeups.md`'s list of unbounded waits gains the owed transition push's retry in `internal/statuscommit/statuscommit.go`: it waits from one second, doubling to a minute while the push stays owed.
