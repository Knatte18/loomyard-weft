```yaml
lens: fix-affected-set
outcome: reported
```

Task: check the fix against the sets the findings name (callers, docs and fixtures under one key). I found no gaps.

No findings.

Sets I rechecked:
- **F2/F4 (`MeasureCall` callers):** the only production caller is `internal/loomcli/darnwiring.go:321`, and it handles the new error return. The other mentions are comments, the stencil, `darnengine` and the `shuttleparity` stand-in. No docs or comments still describe a `--tier 0` fallback.
- **F6 (session's value):** the only user-facing text carrying the env line is `tierrun.go:263` and `gatecli/doc.go:141`, and both now say "tier's value only". No other doc or comment mentions "both values" or "the session has".
- **F1 (TRX and `--results-directory` mentions):** these appear only in the gateslot and gatecli code, tests and docs, and in `docs/dotnet-hub.md`. Other vstest tier declarations are fixtures with no `{args}` (`hubgeom`, `testmap_integration`, `cmd/testtiming`), or the docs tiers already covered.
- **F3 (tier0 tests that spawn):** `copies_test.go`, `redundant_integration_test.go`, `testmap_integration_test.go` and the rest of `tierrun_test.go` still spawn under tier0 or tier1. The review named only the two moved tests and told the fixer to leave the others, so I'm noting them without raising a finding.

Calls made:
1. Grep for fallback, "both values" and "session has" phrasing across `.go`, `.md` and `.yaml`, plus the list of files mentioning TRX or `--results-directory`. My context lacked the repo-wide search for stale wording.
2. Grep for vstest-counter tier fixtures, tier0 tests that spawn, and `MeasureCall(` callers. My context lacked every caller and fixture.
3. Grep for the env-line wording, the measuring-call markers and `MeasureCall` in non-test files, to catch stale docs or comments. My context lacked the doc and comment sweep.
