```yaml
lens: fix-affected-set
outcome: reported
```

Task: check that the fix covers every member of each finding's set (fix-affected-set lens).

List of findings left unchecked: none. I reran the search for F1, F2, F3 and F4.

No findings.

- **F1 and F2:** no plan card still points at the old report config targets in `internal/reportengine` (`Config`, `LoadConfig`, `ConfigTemplate`, `ConfigSpec`, `TestLoadConfig` or `template.yaml`). Card 26 still edits `internal/reportengine#allowedImports`, but only so it gains `internal/raddleparser` (line 35). F1's change doesn't reach that.
- **F3:** among the test files, only `internal/configreg/configreg_test.go` pins the hub-wide module names, in `TestModules_SeedOnlyAndHubWideFlags`, and card 7 now edits it. The package set that loads config with `LoadOrTemplate` is `cmd/lym#configStrictnessDegradingSet`, and card 7 now edits that too.
- **F4:** eight `doc.go` files mention motif. Five of them list the full recipe set: `shedrun`, `loomrecipe`, `shedcli`, `runprofile` and `loomcli`. Card 23 now edits all five. The other three only use motif as an example (`battenshed` and `boardengine`) or concern a motif-only cleanup that the review rejected (`beamcli`). For card 25, `pawlengine/doc.go` is the only file the review named, and it is now under Edit.

Tool calls:
1. Grepped `_lym/plan` and `_lym/loom` for leftover reportengine config refs. My context didn't hold the other plan cards.
2. Grepped the test files for pinned config-module name sets, to find any pinned set beyond the two the review named.
3. Grepped every `doc.go` for "motif", to find all files that list the recipe set.
4. Read card 26's `allowedImports` lines and the motif lines in `battenshed`/`boardengine` doc.go, to judge the matches from calls 1 and 3.
