```yaml
lens: fix-consistency
outcome: reported
```

No findings.

I found one stale sibling mention, below the rubric's LOW bar. `internal/battencli/cli.go:53` says a beam project "skips the draft check". That check now covers draft and discussed, so the comment is slightly narrower than the code. It is still accurate for the beam case it describes, so I did not raise it as a finding.

Other places that name the draft refusal or release agree with the change:
- `startguard.go` field, closure, `refuse` doc and release helper comment all use the new `Unreleased` seam.
- The `startguard_test.go` rows, including the new discussed refusal, use the new field names.
- `grep` finds no remaining `Draft` seam references and no other copy of the "is a draft task" refusal text.
- The README, the board skills and `docs/overview.md` describe only the draft case, as they did before this fix. None of them claim the refusal covers draft only, so nothing contradicts the change.

Further tool calls (2):
1. `grep` across the repo for the old refusal text, `Draft` seam references and the "whose board status is draft" wording: to find twins the context lacked.
2. `grep` of "draft" in `internal/battencli` and `docs`, plus markdown mentions of releasing a draft, then the `cli.go:50-54` lines: to check docs and sibling comments the context lacked.
