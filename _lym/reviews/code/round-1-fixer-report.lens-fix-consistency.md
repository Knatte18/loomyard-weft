```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1/F3/F5/F6 change set for agreement with its surroundings, and for twins left behind.

**BLOCKING.** `internal/battencli/wire_seams_test.go:81`, the twin of the `intentionallyNil` entry added in `internal/loomcli/wiring_seams_test.go:22`.
- **Summary:** F1 removed `Landing.DriftVerdict` from `envkit` `nilLegal`, but battencli's own seam-guard list was not updated, so its guard test should now fail.
- **Evidence:**
  - The grep for `Landing.EnginesRun|Landing.DriftVerdict` finds battencli's list holding `"Landing.EnginesRun": "loom-only seam, batten lands no branch"` and no `Landing.DriftVerdict` entry.
  - That list covers every `Landing.*` field batten leaves nil, so batten's `wire` leaves `Env.Landing` unset, as loomcli's does.
  - `envkit.NilSeams` therefore reports `Landing.DriftVerdict` as nil there, and the test's "nil and not listed" check errors.
- **Why it was missed:** the fixer's gate runs covered `driftengine`, `landingshed`, `loomcli` and `envkit`, not `battencli`.
- **Fix:** add `"Landing.DriftVerdict": "loom-only seam, batten lands no branch"` to the battencli list.
- **Caveat:** I did not read the battencli test body or run it. The failure follows from the grep and the `NilSeams` logic in context.

No other inconsistencies found: the `envkit` stub, the loomcli `intentionallyNil` entry, and the `Deps.DriftVerdict` doc claim ("its drift guard keeps it filled") now agree.

Tool calls:
1. `grep -rn "Landing.EnginesRun\|Landing.DriftVerdict" --include=*.go` over the repo, to find twin seam-guard lists. My context only showed loomcli's list.
