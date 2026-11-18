```yaml
lens: fix-affected-set
outcome: reported
```

**Lens fix-affected-set, list of findings left unchecked:** none. Both findings were rerun with one search each.

**F7 (set: driftengine text describing loomcli behaviour).** I searched the non-test `.go` files under `internal/driftengine` for `amends-plan`, `loom decision`, `loomcli` and PATTERN-change wording. The two refusal lines are gone from `doc.go`. The one hit that is new text is the replacement at `doc.go:276`, which states only driftengine's behaviour.

- **LOW**, `internal/driftengine/doc.go:275` and `:282`. Both lines name `lym loom decision add` and `lym loom decision replace`. They say which parent action is or isn't treated as a judge edit, which is the exemption's own contract rather than the verb's refusal. The review did not flag them and the fix left them alone, so I report no finding.

**F8 (set: kept redundancy-report candidates lacking a keep marker).** The marker is present at `internal/loomcli/driftwiring_test.go:65`, in the same comment block as the other markers in the repo, with the blank `//` separator that `burlercli/cli_test.go` uses. The only other `TestDriftDeps_*` test is the covering test, `TestDriftDeps_RunWhereTheStepsDirResolves`, and it needs no marker. I did not search for other redundancy-report candidates without a marker, because the report is not in my context and the review names only this one.

**Further calls (2 of 6):**
1. `grep` on `internal/driftengine` non-test Go files, to rerun F7's search; my context held only `doc.go` lines 255–299.
2. `grep` for `TestDriftDeps_*` functions plus the marker's references, to rerun F8's search; my context held only the one test.
