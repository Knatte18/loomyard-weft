```yaml
lens: fix-consistency
outcome: reported
```

Lens fix-consistency: check the change set against its sibling docs, tests and cross-references.

LOW — `internal/reportengine/doc.go:46` ("A report walks the records branch's first-parent line once…") and `runs_test.go:238`
- The doc says "walks … once" and then describes widening. The widening re-walks, and the checkpoint walk is a second, separate walk of the records branch.
- The same file's `# Landings` section and `timing_test.go:1596` still speak of one walk. The timing test counts only walks with a path prefix, so it stays correct.
- Evidence: `readCheckpoints` in `landings.go` issues its own `Log` on `src.Records`.
- It is a wording nit, not wrong behaviour. The review already judged the "once" phrasing coherent.

LOW — `doc.go` landings section: "The time join serves … a slug whose pair has checkpoints after its newest tag."
- That text is unchanged and still true.
- It does not state the new limitation: a never-tagged pair's checkpoint older than the parent walk's start is no longer found.
- The checkpoint walk's reach is "the parent walk's start, or the opening of the oldest pair a tag found no task commit for".
- The `fix` key on F5 asked for the old results to be restored. This documented bound is a remaining departure for pairs with no archive tag, so it is worth recording.

No other disagreements found:
- Every `snapshotsOfRun` caller (`runs.go`, `timing.go`) passes `seed.StartedAt`.
- No reader of `inputs.checkpoints` remains.
- `landingInputs` and `selectRuns` docs match the code.
- The `//lym:seam` list matches the structs `typePathTo` still walks, given that `Sources`, `Options` and `Report` are pre-seeded in `seen`.
- No `.md` docs describe the report walks. The grep found only unrelated research reports.

Calls made:
1. `grep -rniE "checkpoint" --include=*.md …` and a `grep` for "walks the records|one walk|same walk" in the reportengine and reportcli Go files, to find stale cross-references outside the files already in context.
