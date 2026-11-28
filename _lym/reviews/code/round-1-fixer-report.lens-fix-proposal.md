```yaml
lens: fix-proposal
outcome: reported
```

Task: fix-proposal lens, challenging the reviewer's proposals that the fix carried out.

**1. MEDIUM: F2, report every dobby's recorded catch-up wait** (`pushsummary.go` `addCatchUp`, `dobbydeps.go` `liveCatchUpWait`, the new `doc.go` sentence "reports every live dobby's wait on origin")

A recorded wait can outlive the owed catch-up, and the summary then reports it indefinitely.
- `catchUpOwed` is held only in memory.
- On a re-exec, `recordStart` returns early at `own && record.StartTicks == d.startTicks` (`loop.go`), so it owes no catch-up.
- `catchUp()` returns early when nothing is owed, so nothing clears `State.CatchUp`.

Case: origin is down, a beam dobby records an origin wait, the lym binary changes and the dobby re-execs, then origin returns. The dobby is live and schedules normally, but its record still says `origin`.
- The prime reports "beam's catch-up from origin for Nm" and posts no all-clear.
- That lasts until some unrelated trigger owes that dobby a catch-up and it succeeds.
- Meanwhile `PushPeriod.Posted` stays true, so later owed pushes get no summary either.

The prime's own record has the same hole. The proposal extends it to every beam, and the live-lock filter does not catch it because the dobby is live.

**2. MEDIUM: F1, "a second origin-wait exit after any successful catch-up is a refused start"** (`starts.go` `oweCatchUpForStart`; the `doc.go` and `summary.md` text for it)

The test cannot tell a failure of the task's own from a second outage.
- Case: origin drops, the first pass exits 80, origin returns and the catch-up succeeds, the task starts again, and origin drops again during that pass. `catchUpSuccesses > stop.successes`, so the dobby posts "dobby could not start the task" at once, backs off and owes no catch-up. On a flapping overnight link this is the report the goal limits to "past the bound".
- `originStops[slug]` is deleted only on claimed or exit 0. Covered exits and other refusals leave it. An unrelated blip days later for the same unchanged entry is therefore refused on its first occurrence, because any intervening catch-up (after a sleep, for instance) raised the counter.

No other findings. F3, F4, F5, F6, F7 and F8 as proposed gave me no breaking case.

Tool calls: none.
