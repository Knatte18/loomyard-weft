# Card 26 — shedrecipe-conformance

**Create:**
- `internal/shedrecipe/conformance_test.go`

**Edit:**
- `internal/shedrecipe/entries_batten_test.go`
- `internal/shedrecipe/entries_simple_test.go`
- `internal/shedrecipe/entries_burler_test.go`
- `internal/shedrecipe/entries_bouncer_test.go`
- `internal/shedrecipe/entries_prrework_test.go`
- `internal/shedrecipe/entries_singlellm_test.go`
- `internal/shedrecipe/entries_discussionwrite_test.go`
- `internal/shedrecipe/entries_planwrite_test.go`
- `internal/shedrecipe/entries_describe_test.go`

**Intent:** Test shedrecipe entry conformance once over the whole registry (D13, #327 §5).
`conformance_test.go` holds `TestEveryEntry_RejectsUnknownConfigKey`, which ranges `Names()` and, for each, builds the entry from a per-name minimal valid config plus one unknown key and asserts the error names that key;
a registry name with no minimal-config row fails the test, so a new entry cannot skip coverage silently.
The per-entry unknown-key tests in the listed files are deleted.
`seedChildEntry` folds into `lifecycleEntryCases()` (filling `Env.SeedChild` through the case's own env builder), and its duplicated `RejectsEmptySlug`/`RejectsBadScratchDir`/`RejectsNilSeams` table goes.
Each entry keeps exactly one `run_subdir` wiring row ("escaping fails") plus "CreatedBeforeAnyCall";
the omitted/empty/absolute rows of the bouncer and burler `RunDirectory` groups go, since `TestResolveUnderRoot` owns them.
The `strings.Contains(err.Error(), field)` checks stay.
Prove it by removing one entry's `configRejectUnknown` call, watching the registry-wide test fail, and restoring it;
name the proof and every deleted test in the commit body.
What this lets through, bounded per D13: an entry gets unknown-key coverage only once it has a minimal-config row, and a missing row fails the test.

**ImpactSummary:** `internal/shedrecipe` tests only; per-entry duplicates collapse into one registry-wide test.
