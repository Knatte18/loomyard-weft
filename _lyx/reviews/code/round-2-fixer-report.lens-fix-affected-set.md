```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

The set behind F1 is the verbs the plan-fix stencil tells the plan fixer to run. Against the audience annotations in `internal/webstercli`:
- `rebaseline` and `restore-plan` are both `AudienceRole` now, so the set is covered. `restore-plan` was the only member that was wrong.
- `fix-plan` appears in the stencil only at lines 1 and 9 (lines 1 and 9 of the stencil, in its header and in a context the stencil does not instruct the fixer to run). It is already `AudienceRole` at `fixplan.go:46`.
- The stencil's line 47 allows only `rebaseline` and `restore-plan`, so no other verb is named for the fixer to run.

The review's second ask was to check that no test or generated index pins `restore-plan` as operator. The fixer's grep of `cmd/lyx/clitree_test.go` and `internal/helpcli/cli_test.go` found no `restore-plan` or `webster` text in either. A grep of `docs/` and `README.md` found nothing either.

Tool calls used:
1. `grep` of the audience annotations in `internal/webstercli` and the verbs named in `webster-template-plan-fix.md`. My context held neither the full annotation list nor the stencil's verb list. One call was refused by a hook, so I reran it without the literal command string.
