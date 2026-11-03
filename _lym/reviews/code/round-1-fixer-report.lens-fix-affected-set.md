```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that the fixes for F1–F5 cover every member of the sets their premises name (sites, callers, docs).

Checked: F1, F3, F2, F4 and F5. Unchecked: none.

**F1 (callers and sites that replace or describe a terminal request).**
- `Submit` is the only replacement site for a terminal request. `hibernatecli/cli.go:105` is its only caller.
- Doc sites updated: `doc.go:27`, `verbs.go:84-85` and the `refuseKeptPause` comment. `doc.go:40` and `verbs.go:180` describe `Cancel` and stay true.
- Finding 1 (LOW): `hibernatecli/cli.go:79-97`, the `request` Long page, says nothing of the new refusal, though it describes the kept pause and its two lift paths. `hibernatecli/doc.go` and `docs/overview.md:489` also say the hub stays paused until `lym tender resume` or `lym hibernate cancel` lifts it, but not that a further `request` is refused meanwhile. This is a stale sibling a reader would notice, but only a mild one. The review did not ask for it, and the Long page is under a 2000-byte cap (commit `74a85d66`), so any addition must fit.
- Finding 2 (LOW): the review's `fix` also asked to "drop the matching Known limits sentence from the change description". `grep -rln "Known limits" _lym` hits `_lym/landing/summary.md`, which sits under `_lym` and is off the fixer's write surface. It still lists the gap as a known limit, so the fixer-report should record it as deferred, not as done.

**F3 and F4 (keep-path sites).**
- The keep branch in `Watch` has one place that finishes a request, and the fix covers it with `recordOwned` before `finish`.
- Every other post-wake exit goes through `failAtWake` or `fail`, which `failRequest` handles and which F2's rows pin.
- No uncovered keep-path site remains.

**F2 (failure call sites of `fail` and `failAtWake`).**
- `fail` is called after the pause, after the confirmation timeout, after the pre-hibernate status read, after a hibernate error or no wake, and after a wake-time lift error.
- The new row pins the pause and hibernate errors with the switch off.
- Finding 3 (LOW): the confirmation-timeout and pre-hibernate status-read call sites have no off-request row. They pass `lift` true through the same `fail`, so the pinned path is the same code. The review's own `fix` named only "the pause or the confirmation", so one of the two is arguably covered.
- Finding 4 (LOW): the `failAtWake` call sites for the reconcile error and for a lift error on the on-path have no off-request row. The advance and status-read sites are pinned, and the reconcile site shares the same call.

**F5.** `chooseResumeOnWake` has one caller, `cli.go:101`, and both new rows exercise it.

**Gate.** The gate passed in your context, with both new `TestChooseResumeOnWake` rows. That confirms `resume_on_wake: [` fails to load.

**Tool calls:**
1. `grep` for "terminal request" and replacement phrasing in `internal`, `docs` and `contracts`: to find every doc and comment stating Submit's replacement behaviour, which my context lacked outside the files I held.
2. `grep` for `Submit` callers and `hibernate request` mentions outside the engine: to find other sites and docs that describe the request flow.
3. `grep` of the hibernatecli integration tests for `request`, `PhaseDone` and `Owned`: to see whether any integration test submits over a finished request that owns a live entry. It found none.
