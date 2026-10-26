```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1 change (restore-plan annotated `AudienceRole`) for agreement with its surroundings.

No findings.

- `restoreplan.go:19` now matches `rebaseline.go:23`, the sibling verb the plan fixer's prompt also names. Both carry `AudienceRole`.
- The other verbs left as `AudienceOperator` (`reset`, `verify`, `status`, `run`, `validate`) are not named by any role prompt in the grep, so they have no twin left stale.
- No test or index file in the grep output pins restore-plan's audience.
- Operator-facing prose that names `lyx webster restore-plan` (the Master stencil at line 252, `validate.go:160`, `run.go:61`) still holds. `rebaseline` already sits in the role index while operator prose names it, so this is the same situation.

Calls made:
1. `grep AudienceAnnotation internal/webstercli/*.go` and `internal/clihelp/*.go`: my context lacked the audience of every sibling verb and the definition of `AudienceRole`.
