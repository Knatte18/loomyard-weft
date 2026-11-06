---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/creelengine/receive.go:331
    summary: noticeDropLocked spends a notice slot and clears the undisclosed count before the notice is posted, so a failed post leaves a drop neither noticed nor counted into a later notice
    fix: In noticeDropLocked, restore window.notices and window.undisclosed to their values from before this call when postDropNoticeLocked returns an error, so the failed notice's drops are counted into the next one and the slot is not spent. Add a row to the existing drop-window test that makes the notice post fail and asserts the next drop's notice counts the earlier drops.
    origin: lens:code-correctness
---

### [LOW] Drop notice slot spent before the notice is posted

**Location:** internal/creelengine/receive.go:331
**Issue:** `noticeDropLocked` increments `window.notices` and resets `window.undisclosed` before `postDropNoticeLocked` runs. When that post fails, the error is only logged. The window has then used one of its five notices and the drops that notice would have counted are lost to later notices. The drop itself stays journaled, so nothing is silent in the journal. The operator's added decision asks that every drop is either noticed or follows a notice that says more may come, and this path can break that on a failed post. The failure needs a journal write error straight after the drop's own write succeeded, so it is rare.

## Rejected

- Lens code-tests finding 1 (no row pins the local-layer refusal of `trusted_hubs`): `internal/configreg/configreg_test.go:204` has the row `{"creel", "trusted_hubs:\n  - ly\n", "trusted_hubs", false}` in `TestModules_LocalLayerRefusals`, which pins it.
- Lens code-tests finding 2 (`post --interrupt` refusal may be unpinned): `internal/creelengine/verbs_test.go:843` includes `PostInterrupt` in the row `AnotherHubsNameIsRefusedByEveryVerbThatStaysInsideOneHub`.
- Lens code-correctness finding 2 (`Scheduler.Stop` can wait up to the 30 s call timeout for an in-flight `deliver`): the wait is bounded and matches Stop's existing contract of waiting for sends in flight, and D2 sets the 30 s timeout; no defect.
- Daemon re-recording its hub at a served local verb (`hubs.go` `recordHub`) is beyond the decision record's "records at start": the plan licenses it in `_lym/plan/00-overview.md:42` and card 4, so it is not a scope finding.
