```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

Checked:
- F1: no live `lyx loom review approve` remains in Go, Markdown or YAML outside `_lyx` and `docs/research`. `docs/research/twill-fork-timing-bench.md:150` is a historical measurement report and is exempt. The other `loom review` hits mean "loom review segment/perch", not the removed verb. `internal/loomcli/circling.go:8` and `cli_test.go:34` mention the removed `review` verb, and both are correct as they stand.
- F2: no `AlignmentDir` or `AlignmentRel` reference remains anywhere. The remaining "alignment" hits in `reportengine` are the D4 archived-record decoders and their fixtures. `shedrun.RunTreeRels.Alignment` and `alignmentDirName` stay because `internal/reportengine/wakes.go:228` reads them.

F3 is a coverage-gap finding, not a set, so I did not rerun a search for it.

Tool calls (1):
1. One `grep` over Go, Markdown and YAML for F1 and F2 together. My context showed the edits but not the repository-wide state after them.
