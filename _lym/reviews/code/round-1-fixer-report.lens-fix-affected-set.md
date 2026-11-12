```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: set-premise findings (F3, F6, F2) rechecked.

- **LOW (F6, twin left unchanged):** `internal/gatecli/testmap.go:375`, the `gate testmap` module-error way forward. It still reads "set the dir of the lowest of tier0 and tier1 to the Go module's root…". F6's fix reworded only the `redundant.go` copy to "declare tier0 or tier1 with its dir at the Go module's root…". Both calls reach the same `redundancyModule` failure path, so the two verbs now word the same step differently. The no-tier cause F6 named applies to `testmap.go` equally. Worth aligning only if the shared wording is wanted.
- **F3 covered:** the pilot report is now listed, and `coverage-map-pilot-2026-11-12` is the only new unlisted entry the diff introduced. These seven `speed/` entries were already unindexed before this round and are outside the finding: `fabricengine-per-test-times-2026-11-07.md`, `fabricengine-shards-2026-11-09`, `fabricengine-spawns-2026-11-09.md`, `fabric-test-cuts-remeasure-2026-11-08.md`, `fabric-test-redundancy-2026-11-08`, `fabric-test-speed-remeasure-2026-11-07.md` and `test-targets-bench-2026-11-07.md`. The review's premise that the index "lists every report under `speed/`" is false as written, but the fix covers its stated ask.
- **F2 covered:** `map_commit` and `map_built_at` appear in non-test, non-verb code only in `redundancyreport.go`, a different report with its own struct. Nothing else describes this verb's answer keys, so no doc or consumer of the `redundant` answer was left without the new `map_build_failure` key. In that other path, `redundancy.go:306` refuses on `BuildFailure` and does not read `Failure`. That is outside this change.
- **Unchecked:** F1, F4, F5 and F7 are not set-premise findings.

Tool calls:
1. Bash: listed `speed/` against the README index, and searched for the F6 wording and the F2 key names. Context lacked the directory listing and the other copies of the strings.
2. Bash: searched `Failure` handling in the sibling redundancy report code. Context lacked whether a twin of the new key exists.
