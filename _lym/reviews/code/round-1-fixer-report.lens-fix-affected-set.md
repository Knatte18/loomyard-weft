```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Set premises checked: F1 (every user of the stall timer seam) and F5 (every test that installs it).
- `startStallTimer` has one production call site, `gitexec.go:301`. The new default-timer row calls the captured default and covers it.
- `SetStallCheckForTest` has three callers, all in `TestRun_RemoteBounds`. F2, F3 and F4 cover all three: the silent row (F2), the unarmed row (F4) and the answering row (F3).
- `SetStallBoundForTest` has the same three callers.
- No other test installs the seam, so the F5 serial-state comment names every row that sets it.
- Findings left unchecked: F2, F3 and F4. Each concerns one row, not a set.

Tool calls:
1. `grep -rnE` for the seam's symbols over `.go` and `.md` files outside `_lym`. My context lacked a repo-wide list of users of the seam.
