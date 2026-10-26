```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the F1 change (restore-plan annotation switched to `AudienceRole`) for code correctness.

The change is one constant in `restorePlanCmd`'s `Annotations` map, `clihelp.AudienceOperator` → `clihelp.AudienceRole`. The function has no new branch, signature change or new test.

Walking one input through it: `restorePlanCmd()` builds the cobra command with `Annotations[clihelp.AudienceAnnotation] = AudienceRole`. The command index lists the command under its annotated audience, so `restore-plan` now appears in the role index, as `rebaseline` does. The `RunE` body is untouched, so a refusal or failure of any call it makes behaves as before.

Callers and pins:
- My context's grep over `*.go`, `*.md`, `*.txt` and `*.golden` found no test or doc outside `restoreplan.go` that pins `restore-plan` to the operator audience.
- The only audience-aware files in the repo are `cmd/lyx/clitree_test.go`, `internal/helpcli/cli.go` and `internal/helpcli/cli_test.go`. The grep for `restore-plan|webster` in `clitree_test.go` and `cli_test.go` returned nothing, so neither pins the verb.
- The fixer's gate run over `./internal/webstercli ./cmd/lyx ./internal/helpcli` was green (tier0, 491 passed, 0 failed).
- No test was added, which matches the rule that a non-coverage finding adds none.

no findings

Tool calls made: none; my context already held the diff, the grep results and the gate output.
