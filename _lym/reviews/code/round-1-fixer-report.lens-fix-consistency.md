```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1–F3 change set for consistency with its surroundings (lens fix-consistency).

Findings:

- **LOW** — `internal/boardengine/projectrelease_test.go`, doc comment of `TestSetStatusReleasingProject_Refusals`. The comment says a run-status or done project "names each draft member's release as the way forward". The new `DoneProjectWithoutDraftMember` row covers the other branch, where the error says there is no draft member to release. The comment no longer describes every row. This is a small gap a reader would notice, not a wrong behaviour.

Everything else agrees:
- `stepProjectRelease` is still wired in `cli_test.go:149`, and its two-line comment now sits directly above its func (`verbs_test.go:546-548`).
- The `stepProjects` comment block was moved back above its func.
- The renamed row `RunStatusProjectNamesEveryMember` has no other reference in `internal`.
- The `OtherWrites` doc comment now lists the added non-null and task cases.
- The new `fmt` import in `projectrelease_test.go` is used.
- The `NullStatusOnTask` expectation of `""` for `p` matches `statusOf` returning `""` for an empty-string status, and the gate run passed.

Tool calls:
1. `git show --stat` plus a grep for `stepProjectRelease` and `RunStatusProject`, to check the F1 commit's scope and look for stale references to the renamed row. My context held the files but not the commit or the callers.
