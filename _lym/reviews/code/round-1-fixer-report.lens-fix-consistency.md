```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F2/F3/F4 change set for agreement with its surroundings.

Findings:

1. **LOW** — `internal/beamcli/deps.go:199` (the new `FileTask` release error). It names the partial state, but not that the stranded draft will make a rerun refuse. On rerun, `openMergeTask` (`mergeparent.go:94-100`) finds the draft by project and slug suffix, because `IsDoneStatus` is false for a draft. `mergeParent` then returns "stands open" with a way forward of `lym batten status` or `lym batten cancel --remove`, which targets a run the draft never had. That is the misleading way forward F4 aimed to remove.
   - The new message says "remove it with `lym board` or release it, then retry". That works only if the operator follows it before rerunning.
   - It also names a bare `lym board` where sibling errors in `create.go:108-112` and `land.go:186-188` name a concrete subcommand (`lym board upsert`, `lym board list`). A removal command would match that convention.
   - The rerun refusal stays as the finding described.

No other disagreements:
- The F3 swap to `boardengine.IsDoneStatus` is semantically identical to the old literal check.
- The test's `done := boardengine.StatusDone` still yields a `*string` that `Status: &done` takes.
- The new F2 test uses the `landingHub` seed label `bug`, as the seed does.
- The file headers of `deps.go` and `dobbydeps_integration_test.go` still describe the seams the change touches.

Further tool calls:
1. `git diff HEAD~3 -- deps.go mergeparent.go` plus a grep for "lym board" in `internal/beamcli`. My context showed the pre-format edits only, and I needed the committed form and the sibling `lym board` wording.
