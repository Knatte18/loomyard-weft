```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Per-finding set check:
- **F5** (budget sites): the repo-wide search for `MessageLine` and `boundaryClaimBudget` shows one budget site, `ClaimAtBoundary` (`verbs.go:187-191`), which the fix changed. The one other `MessageLine` consumer that joins lines is `internal/creelcli/hookread.go:69-71`. It joins with a single `"\n"`, which the fix now counts. The three places that state the budget (the const comment, the method doc, `doc.go:111`) all name the newlines. `doc.go:112` and `creelcli/doc.go:22` don't state the budget, so nothing there is stale.
- **F1** (`Prepare` implementations): the only other `Prepare` is `shuttlefake`, a test fake. It writes no settings and has no hook to wire. `claudeengine` was the only implementation that needed the new test.
- **F3** (`Idle` callers): `SetIdleHook` is new and unset by default, so every existing user of the fake `Idle` behaves as before. The fake's other methods lock per call, so unlocking around the hook can't deadlock.
- **F2, F4**: the premises name single sites (the `Run` reconcile, the flags table return row), and the fixes cover them. I did not check them further.

Tool calls:
1. `grep` for `MessageLine`, `boundaryClaimBudget` and the 8 KiB wording across `.go` and `.md` files. My context lacked the repo-wide list of budget sites and the `hookRead` call site.
2. `grep` for `Prepare(` implementations and `Idle(` uses in `testkit`. My context lacked the set of `Prepare` implementors and of fake `Idle` users.
3. `sed` on `internal/creelcli/hookread.go` lines 55-95. My context lacked how `hookRead` joins the claimed lines.
