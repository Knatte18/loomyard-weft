```yaml
lens: code-tests
outcome: reported
```

**Lens: code-tests.** Per behaviour, the test that pins it, or none.

| Behaviour | Pinning test | Result |
|---|---|---|
| Limited held turn end sends no notice, leaves `NotifiedOffset` at 0, holds to the deadline | `hold_test.go` row "a usage-limited stop sends no notice, leaves the offset and holds to the deadline" | Pinned. |
| A later non-limited held turn end is notified | `hold_test.go` row "a stop after a usage-limited one is notified" | Pinned. |
| Deferred run sends no notice for a limited stop, even after the deferral | `hold_test.go` row "a deferred run sends no notice for a usage-limited stop after the deferral" | Pinned. It would fail if the `usageLimited` case sat after the `HoldDeferral` cases. The predicate returning false is the same shape as the existing row at `hold_test.go:337`, which notifies after the deferral. |
| An engine without `UsageLimitReader` reports no limit | None added. | Covered implicitly: every existing row runs on `fakeEngine`, which lacks the capability, and still gets notified. |
| `heldTurnEnd.raw` set at the `pollEventsTick` turn-end site (`wait.go:872`) | The limited-stop rows | Pinned, since the fake matches on `Raw`. |
| `heldTurnEnd.raw` set at the `EventWaiting` site (`wait.go:851`) | None. | Gap: no limited row reaches `waitingTurnEnd`. The limited rows all use `STOP:`. |
| Claude: main-chain `rate_limit` turn end is limited | `activity_test.go` row "a main-chain usage-limit turn end is limited" | Pinned. |
| Claude: `server_error` turn end is not limited | `activity_test.go` row "a main-chain server-error turn end is not limited" | Pinned. |
| Claude: plain turn end is not limited | `activity_test.go` row "a plain turn end with no fork is not limited" | Pinned. |
| Claude: fork stopped on the limit during the turn makes the turn limited | `activity_test.go` row "a fork stopped on the limit during the turn makes the agent's own turn end limited" | Pinned. |
| Claude: fork written before the previous turn end does not count | `activity_test.go` row "a fork stopped on the limit before the previous turn end does not count" | Pinned. |
| Claude: a fork's newest assistant entry is what counts | `activity_test.go` row "a fork that answered after a limit entry does not count" | Pinned. |
| Claude: fork on `server_error` does not count | `activity_test.go` row "a fork stopped on a server error does not count" | Pinned. |
| Claude: no previous turn end counts every fork | `activity_test.go` row "a first turn counts every fork" | Pinned. |
| Claude: missing `transcript_path` or missing file reads as no limit | `activity_test.go` subtest "a missing transcript path or file is not limited" | Pinned. |
| Claude: an older limit entry, with a plain newest turn end, is not limited (the "newest wins" direction in `readTurnLimit`) | None. | Gap. No row has a limit entry followed by a plain turn end. A scan that took any limit entry in the tail would still pass every row. |
| Claude: a sidechain entry in the main transcript is excluded from the turn-end pick | None. | Gap. |
| Claude: an unparsable previous-turn-end timestamp falls back to the zero time | None. | Gap. |
| Claude: `readBackward` chunk boundaries | None added. | Not an economy issue. `readBackward` is an existing helper, and its callback has no growth case here. |

**Test economy:**
- No new test duplicates existing coverage. The hold rows and Claude rows each pin a distinct behaviour.
- No per-helper tests were added.
- No stencil-prose assertions were added.
- The three hold rows extend `TestWait_HeldTurnEndNotice`, and the Claude cases are one table-driven test.

**Findings:**

1. **LOW / scope** — `internal/shuttleengine/claudeengine/activity_test.go` (`TestTurnEndUsageLimited` table, near line 190). No row covers an earlier usage-limit main-chain entry followed by a plain newest turn end. The "newest main-chain turn end decides" behaviour, which keeps a limit an earlier turn already reported from silencing a later notice, is therefore unpinned. Evidence: the table's main-chain rows end with `limitTurnEnd`, `serverTurnEnd` or `agentTurnEnd` after only `earlierTurnEnd`, and none puts `limitTurnEnd` before `agentTurnEnd`. Fix: add a row `{limitTurnEnd, agentTurnEnd}` with no fork, want false.

2. **LOW / scope** — `internal/shuttleengine/claudeengine/activity_test.go`. The sidechain-exclusion and unparsable-previous-timestamp branches of `readTurnLimit` have no row. Evidence: `readTurnLimit` has the `e.IsSidechain` filter and the discarded `time.Parse` error, and no row exercises either. Fix: add two rows. One has a sidechain `rate_limit` entry as the newest main-transcript line, want false. The other has a previous turn end with a bad timestamp and a pre-turn fork, want true.

3. **LOW / scope** — `internal/shuttleengine/hold_test.go`. The `raw` set on `run.waitingTurnEnd` at `wait.go:851` is not exercised by any limited row. Evidence: only the `STOP:limit` rows exist, and they reach the `held` site at `wait.go:872`. Fix: add a `WAIT:`-style limited row only if `waitingTurnEnd` can reach `notifyHeld` or `usageLimited`. This is unconfirmed, because I did not read the consumers of `waitingTurnEnd`.

**Tool calls:**
1. `grep` of `hold_test.go` and `wait.go` for the deferral predicate and `pendingHold` usage. My context lacked the semantics of the `deferral` closure and the existing row at `hold_test.go:337`, which I needed to judge whether the deferred row discriminates.
