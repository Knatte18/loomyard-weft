```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix change set through the fix-consistency lens, meaning siblings and twins that should agree with the changed text.

No findings.

- The `docs/overview.md` claim that a done run that isn't a motif run "is still refused, naming `lym batten cleanup --apply`" matches `internal/battencli/arm.go:880` and `internal/beamcli/doc.go:27`.
- The `ClearMotifRun` doc says a failed removal writes the status and child-recipe records back. That agrees with `restoreMotifRunRecords` and with `create.go`'s "fix the cause it names, then run `lym beam create` again" way forward, so a re-run now finds the run.
- The `removeRunRecords` doc, which says a removal whose commit failed finishes on a re-run, still holds for `cleanup`. `clearMotifRun` now keeps its own retry path.
- The `beamcli/create_test.go` rows that fake `ClearMotifRun` are untouched, because `createSteps` did not change.

Tool calls:
1. `grep -rn "batten cleanup --apply"` over `internal` (non-test Go files): to check that the overview's new "refused, naming `lym batten cleanup --apply`" sentence is backed by the code. My context only had the review's claim.
