```yaml
lens: fix-consistency
outcome: reported
```

No findings.

Task: fix-consistency lens over the F1–F3 change set.

Checked for twins left behind:
- **F1:** the same removed verb was in the `resume.go` refusal message. The fixer changed it too. The only remaining `lyx loom review approve` is in `docs/research/twill-fork-timing-bench.md:150`. That is a measurement report of one past run, so the stale verb is expected there.
- **F2:** no `AlignmentDir` or `AlignmentRel` reference remains in code or docs. `shedrun/doc.go`, the `wiring.go` doc comment, `paths_test.go` and `wiring_commitstatus_test.go` are all updated. `alignmentDirName` and `RunTreeRels.Alignment` are rightly kept, because `reportengine/wakes.go` still reads them. The `wantAligned` field and its test row are gone together.
- **F3:** the new test only reads `archivedrecords.go` symbols, so no sibling is affected.

Tool calls (1):
1. A single `grep` pass over the repo for the removed symbols, the stale verb and the "alignment" mentions in `wiring.go` and `doc.go`. My context did not show the whole repo, so I could not confirm the absence of leftover references otherwise.
