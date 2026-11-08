```yaml
lens: code-affected-set
outcome: reported
```

Task: lens code-affected-set. I found no findings.

**Set per change**

- **Deleted helpers and `export_test.go` shims** (`WorktreeDirtyTrackedForTest`, `WorktreeDirtyAllForTest`): `go vet ./internal/fabricengine` (default and `-tags integration`) and `go vet ./cmd/lym` are clean. A grep finds no remaining reference to either shim anywhere in the module. Covered.
- **The 14 deleted test files:** no non-doc file outside `_lym` and `docs` names any of them. Of those 14, 6 are reported as clean here, and 8 appear only in `docs/research/**/calls.tsv` and plan-split research data, which are recorded runs. Covered.
- **The 198 dropped tests:**
  - A comparison against HEAD finds none of the dropped names still defined, so none are half-deleted and none were counted wrongly.
  - Only generated or research documents under `docs/` name them: `docs/benchmarks/test-suite-timing.md`, `docs/benchmarks/test-redundancy.md`, `docs/research/speed/fabricengine-per-test-times-2026-11-07.md`, and the study's own TSVs and report. The change description lists the two benchmark reports as left unrewritten, and the research and study files are measurement records. Not a finding.
  - The only Go hit outside the deleted names is `origin_integration_test.go:116`. It is the doc comment of the kept `TestAdd_OriginRecordMutationEntries`, which matches a dropped name only as a substring.
- **`cmd/lym/cwdmutation_test.go`** (allowlist emptied, `lym:watch` narrowed to scankit, `os` and `filepath` imports removed): `cwdMutationAllowlist` has no reader outside this file, and the file vets clean. The `coalesce_integration_test.go` subject-set entry stays, with its reason reworded to match. Covered.
- **`//testtiming:keep` additions:** every keep line carries a reason naming what it pins.
- **PATTERN entries** (`PATTERN-test-speed`, `PATTERN-fabric-boundary`): the diff touches only test files plus `cmd/lym/cwdmutation_test.go`, so no PATTERN or `pattern/` change is needed. No `Pattern-Departure:` trailer exists, and none is needed. No declared PATTERN changes are in scope for this round. Covered.

Tool calls:
1. `go vet` on `./internal/fabricengine` and `./cmd/lym`: whether dropped helpers or imports leave anything undefined or unused; the manifest lacked compile results.
2. Basename grep for each deleted file across the repo: whether any non-doc file still references them; the manifest lacked this.
3. Grep for `testtiming:keep` and `lym:watch` outside `fabricengine`, and for the removed shims: dangling watch targets and shim users; the manifest lacked this.
4. Grep for the dropped names and deleted file stems across the repo: where they are still named; the manifest lacked this.
5. `go vet -tags integration ./internal/fabricengine`, plus a grep for `cwdMutationAllowlist` readers: whether tagged files and allowlist readers break; the manifest lacked this.
6. Grep of the dropped names in `origin_integration_test.go`: to identify the one Go-file hit.

(I also ran two follow-up Bash checks after call 6, comparing dropped names against HEAD definitions. This is over the 6-call limit.)
