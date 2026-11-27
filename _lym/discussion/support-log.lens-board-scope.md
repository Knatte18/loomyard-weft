```yaml
lens: board-scope
outcome: reported
```

**Lens: board-scope.** I checked the record's fit with the board: the entry's goal points, the entries that touch it and the peers.

**Findings**

1. MEDIUM · Scope (Out), D2 network row, and the entry's first stop ("A start failed on 'push lock busy' or 'held board lock'").
   - **Summary:** the record never says which side owns "held board lock" at a dobby start.
   - **Evidence:** `push-retry-not-halt` D8 covers only a held push lock at the catch-up. The record's D4 covers a board claim refused at `Worktree-Create`.
   - **The gap:** a dobby start that fails on a held board lock, and needs `lym board sync` plus `lym batten run`, is a third shape. No decision ties it to either side, so the entry's point may be half met.
   - **Fix:** state that this failure is the `Worktree-Create` board claim class (D4), or add it to D4 as a start-time class.

2. MEDIUM · D10 / D7, and the entry's "an escalation the orch had answered without releasing the run".
   - **Summary:** D10 chases only runs handed back with `settle: orch`.
   - **Evidence:** an escalation the arbiter hands on, or a tackler hand-back, carries `settle: orch` and is covered. A stop past lym's bound whose class the table leaves unnamed does not obviously carry it.
   - **Fix:** say that every hand-back to the orch carries `settle: orch`, so all of them are chased.

3. LOW · Open risks, `worktree-create-transient-net`.
   - **Summary:** that entry also changes Worktree-Create's stop handling, and the record does not name it in Open risks.
   - **Evidence:** it depends on `push-retry-not-halt` and asks Worktree-Create to retry and to fix its stop question. D4's new board-claim row sits at the same row, and Scope (Out) names that entry.
   - **Fix:** add a line saying the raddle promotion names it in `depends_on`, or that the two rows' classes do not overlap.

4. LOW · Open risks, `beam-member-parent`.
   - **Summary:** the record says D7's and D10's posts follow the run's recorded parent, but D7's Slack post routes by the program sender's shortname.
   - **Evidence:** `internal/slackengine/outbound.go` sends a `<shortname>:<NAME>` program to the hub orch's channel, and `<shortname>:<slug>:<NAME>` to the beam's orch. Before `beam-member-parent` lands, a beam member's severe post reaches the hub orch's channel.
   - **Fix:** state which program name D7 posts under, the beam's program for a beam member, or accept the hub channel. Nothing contradicts; this is a clarification.

**No conflict found**
- No `Decision for self-healing-runs:` note exists on the board.
- `push-retry-not-halt` (Parent-Align): the network, push, catch-up and landing classes are left to it, and D4's pause wait reuses its D6 wait record. Consistent.
- `kde-window-placement-build` and `pr-session-test-stand-ins`: no overlap.
- The `battencli-split-long-tests` entry, the `fabric-core-*` entries, and the `keep-landed-run-records` entry (hard-cap open findings kept in records): no conflict.
- Commit `9acc400f8` (motif and raddle) already makes the orch post split-ready to Slack and never approve the split. D8 keeps that line quiet with no mention, which fits it.
- `lym loom peers` lists no parent commits.

**Goal points of the entry**

| Goal point | Met? |
|---|---|
| Mechanical stops are retried with a bound and only unsettled ones reported | met (D4, D5, D6, peer) |
| A rule table per stop class decides who settles, severe only to the operator | met (D2, D7) |
| Quiet notices name what the agent waits on | met (D11) |
| The study's three open points are settled | met (D3, D6, D8) |
| Overnight, nothing non-severe reaches the operator | met (D9) |
| Severe halts wait, reach Slack as imperative messages answerable from the phone, other work goes on | met (D7, D8) |
| A stricter second level, not named "full auto" | met (D1, D13) |

The entry's five hand-settled stops:

| Stop | Met? |
|---|---|
| `driver_not_parked` | met (D5) |
| Push lock or held board lock at start | partial (finding 1) |
| Budget round with only deferred findings | met (D6) |
| Hard cap before Code-Review | met (the arbiter accepts, as today) |
| Idle notices for correct waits | met (D11) |

**Tool calls**
- One call read the `worktree-create-transient-net` and `beam-member-parent` board entries. My context held only their titles.
