```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix change set for twins left stale (fix-consistency lens).

Findings:

1. **LOW**: `contracts/stencils/stencils.go:54`, the comment "The writer and its subagents run no gate in it." is the twin of the exception F7 added in `darn-fragment-make-change-gated.md:2`, `internal/darnengine/doc.go:79` and `internal/darnengine/spec.go:69,81`, and it was left unchanged. It now contradicts step 3 of the fragment. The review named only the fragment, `doc.go` and `spec.go`, so the fix followed it, but the rule is the same one stated in a fourth place.

2. **LOW**: `internal/darnengine/spec_test.go:110`, the case name "a cold fixer on the verify gate has the writer and its subagents run no gate" now overstates the stencil. The change is not wrong, and the name only describes the assertion. I did not read the assertions, so I cannot say whether they need an exception row.

I found no other inconsistencies:
- No other text keeps the old "has nothing to run and passes" wording; the grep matched only the changed lines.
- The `--tier <n>` hits in `loom-plan-checks-spec.md`, `verifymodule.go`, `webster-body-implementer.md` and PATTERN are for webster and plan cards, not the darn writer, so they are out of scope.
- The `gatecli/cli.go` help, the `doc.go` sentence and the template comments agree with each other and with `allScope`'s doc comment.

Tool calls:
1. One `grep -rn` over the repo for the old wording and the stale rules. My context holds only the files the fixer read, not the repo's other mentions.
