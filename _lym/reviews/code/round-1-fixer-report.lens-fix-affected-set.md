```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun the set searches for F1 (`ChangedPaths` consumers and seam lists), F3 (landed-commit writers) and F5 (`report_dir` sites), and check the fix covers every member.

Findings:

- **LOW** — `internal/darnengine/doc.go:16`, a stale sibling of the F5 fix. It says "LoadConfig refuses a `report_dir` that leaves the worktree." `internal/darnengine/config.go:111` now also refuses a path under `_lym` or `.lym`. A reader of the package doc would not learn about the second refusal. The fix is to extend that sentence.

No other findings:

- **F1:** `ChangedPaths` consumers are `parentalign.go` and the `finalize.go` merge-in checks, none of which reads the landed diff. The only seam-list sites are `loomcli/wiring_seams_test.go` and `battencli/wire_seams_test.go`, and both now carry `Landing.ChangedCodePaths`. The `landingdeps.go` wiring is added. `publish_integration_test.go:52` sets only `ChangedPaths`, so it leaves `ChangedCodePaths` nil, which means "not empty". That is harmless, since it asserts nothing about the flag.
- **F3:** From the earlier grep, `RecordLanding` is wired only in `landingdeps.go`, now through `RecordLandedCommit`. The only other `WriteLandedCommit` caller is the `beamcli` test, which writes a fixture directly and is correct.
- **F5:** Beyond the `doc.go` sentence above, the only other `report_dir` sites are `template.yaml` and the stencils. They need no change.
- **Not covered by the finding itself:** a one-repository hub with a non-root anchor keeps records at `<anchor>/_lym`, for example `backend/_lym`. F5's check covers only the root names that `lymdirs` declares. That is the scope the review asked for, so I record it only as an observation.

Calls:
1. Ran grep over `.go` and `.md` files for `ChangedPaths`. The context lacked the full set of consumers and seam-list sites outside `fabricengine`.
2. Ran grep over yaml, md and go files for `report_dir`. The context lacked the sites that describe the key, such as `doc.go` and the template.
