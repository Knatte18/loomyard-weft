```yaml
lens: fix-consistency
outcome: reported
```

Task: check the change set's agreement with its surroundings (fix-consistency lens).

**Findings**

1. LOW — `internal/gatecli/test_integration_test.go:1699`, `internal/gatecli/tiers_test.go:233`, `tiers_test.go:299` and `tiers_test.go:461`.
   - The test names still say "without {targets}" or "lacks {targets}".
   - The message and behaviour they pin now speak of "neither {targets} nor {test_targets}".
   - The assertions still pass, so these are stale test names, not failures.
   - Evidence: `test_integration_test.go:1699` has `"a command without {targets} runs whole under explicit targets"`, and its assertion at line 1707 was updated to the new message.

2. LOW — `docs/dotnet-hub.md:16`.
   - It says "A tier whose command holds `{targets}` then runs once per selected test project."
   - The dotnet narrowed-share path goes through `HoldsTargets`, which now also accepts `{test_targets}`.
   - The wording is the same `{targets}`-only twin that finding F3 was about. Whether a dotnet command can usefully hold `{test_targets}` is unclear, so this is cosmetic.

Everything else I checked agrees:
- The updated message at `run.go:486` and its test assertion.
- The guard-step refusal in `tiers.go:165` and both `tiers_test.go` expectations.
- `gateslot/tiers.go:187` and `config_test.go:311-312`.
- `template.yaml:7`, which already names both placeholders.
- The new comment and doc wording in `tiers.go`, `gatecli/doc.go` and `gateslot/doc.go`.

Calls made:
1. Ran one `grep` over `*.go`, `*.md` and `*.yaml` for `{targets}`-only wording. My context did not show the twin sites in files the fixer never opened: `docs/dotnet-hub.md`, `template.yaml`, `config_test.go`, and the test names.
