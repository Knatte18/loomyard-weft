```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I name the test that pins it, or none.

**Behaviour → test that pins it**

`internal/retrocli/cli.go`
1. `parseWindow` refuses named slugs given with `--since` or `--until`: pinned by `TestRetro_RefusesBadArguments` rows "named runs with a since/an until".
2. `parseWindow` refuses an invalid `--since` or `--until` date: pinned by rows "a since that is no date" and "an until with no such month".
3. `parseWindow` refuses `--max-runs` &lt; 1: pinned by row "a cap of zero runs".
4. `parseWindow` puts a valid date into the right field, `--since`→`Since` and `--until`→`Until`, at UTC midnight: none.
5. `resolvePrime` refuses a worktree that is not the prime: none.
6. `fileRetro` refuses when `tools/retro` is missing: none.
7. `reportDirNames` reads an absent directory as no names, and `fileRetro` merges the report-dir names with the board slugs for `LastRetro`: none.
8. `landedRuns` drops landings with no slug and maps `Committed` to `Run.Landed`: none.
9. The duplicate board slug is refused: none.
10. The task is filed with `UpsertTask` and released with `SetStatus(nil)`, with a draft-left error when the release fails: none.
11. The success envelope carries `slug`, `runs`, `from`, `until` and `report_dir`: none.
12. The board write carries the strand name from `LYM_STRAND_NAME`: none.

`internal/retroengine/retro.go`
13. `LastRetro` (none found, foreign or malformed names, newest wins): pinned by `TestLastRetro`.
14. `Window.Start`, default, last-retro and since branches: pinned through `TestSelect` rows 1–3.
15. `Window.Start`, named branch (`Now - NamedLookback`): none. No row asserts the date in the "no landing since …" error.
16. `Select` keeps each run at its newest landing: pinned by row 1 (`relanded`).
17. `Select`'s window bounds, start exclusive and end inclusive: pinned by row 3 (`a` at 10:00 = Until) and the "no landed run" row (`relanded` at 12:00 = last).
18. `Select` keeps the oldest runs past the cap and moves `Until`: pinned by the "past the cap" row.
19. `Select` covers named runs exactly, dedups names, sorts by landing, and takes the stamp from the newest run: pinned by the "named runs" row.
20. `Select` refuses an unlanded name, too many names, and an empty window: pinned by the three refusal rows.
21. `sortByLanding` breaks ties by slug when two runs land at the same time: none, because no row has two runs at the same landing time.
22. `Plan.Body` fills every marker of the shipped stencil: pinned by `TestPlan_BodyFillsTheShippedStencil`.
23. `Plan.TaskFields` sets slug, title, `kind=task`, `recipe=darn`, `profile=docs` and `labels=[research]`: none.

`contracts/stencils/stencils.go` registration: pinned by `TestPlan_BodyFillsTheShippedStencil` through `Registry().Default`.

`cmd/lym/main.go` mount: pinned by the existing `cmd/lym/registration_test.go` and `clitree_test.go` oracles. The `sandbox_coverage_test.go` row is pinned by that test itself.

`reportcli` `PrimeGeometry` rename: no behaviour change; the existing report command tests and the compiler hold it.

`tools/retro`
24. Per-session turns, calls, lookup turns, serial count, refusals, gate runs and failures, and the usage join for row, model and cost on a main session and a fork: pinned by the `sessions.jsonl` assertion of `TestRetroScript_WritesTheFiguresOfEachRun`.
25. The refusal row: pinned by the `refusals.jsonl` assertion.
26. The `rows.jsonl` verify runs and reworks: pinned.
27. Reading only the newest facts file of each perch: pinned through the figures totals ("Findings 4" and "design 3", where the older file would give 9).
28. The recurring keys, read only under the Recurring heading: pinned by `keys.jsonl`.
29. `missing.jsonl` for a missing archive tag and a missing transcript dir: pinned.
30. The figures Runs row: pinned.
31. The "Producers against their medians" table, its medians and its outlier rule (≥3 sessions, ≥2× median): none.
32. Gate failure read from a background task notification, or from a `"done":false` result: none.
33. The "Gate-fixer sessions" column (the title suffix `:gate-fixer`): none, since the fixture count is 0 and a broken match still yields 0.
34. The rendering of the serial-lookup, refused-command summary and agent-gate tables: none; only their jsonl inputs are pinned.
35. The default `HUB` and `RECORDS` derivation from the prime worktree: none, since the test sets both.

**Findings**

1. MEDIUM, design. `internal/retrocli/cli.go:360-417` (`fileRetro`, `landedRuns`, `reportDirNames`, `openBoard`). The verb's main path has no test row: filing plus release, the release-failure message, the duplicate-slug refusal, the missing-tools refusal, the non-prime refusal, merging report-dir and board names for the last retro, dropping slugless landings, and the success envelope. Evidence: `internal/retrocli/cli_test.go:1-46` covers only pre-resolution argument refusals; no other test calls `retrocli.Command` past `parseWindow`.
2. MEDIUM, design. `internal/retroengine/retro.go:838-849` (`Plan.TaskFields`). The board fields that make it "one darn `docs` task" (recipe darn, profile docs, kind task, research label, slug) are untested. This is a pure tier0 function. Evidence: `internal/retroengine/retro_test.go` has no reference to `TaskFields`.
3. MEDIUM, design. `tools/retro/figures.jq:25-33`. The board entry names "turns and cost per producer against their medians" as a figure, and no test pins the median computation or the outlier rule. Evidence: the figures assertions in `tools/retro/retro_integration_test.go:192-197` cover only the Runs row, the column totals and the ledger line.
4. LOW, design. `internal/retrocli/cli.go:342-355`. No test pins the success path of `parseWindow`: `--since` landing in `Window.Since` and `--until` in `Window.Until`, as UTC dates. Swapping the two `into` pointers fails no test. Evidence: `cli_test.go:24-28` has refusal rows only.
5. LOW, design. `tools/retro/transcript.jq:55-62`. No transcript fixture covers the background gate-failure path (task-notification exit code) or the `"done":false` result. Evidence: the `retro_integration_test.go:142-149` fixture has one foreground `Exit code 1` gate result only.
6. LOW, design. `tools/retro/figures.jq:24`. The "Gate-fixer sessions" count (title `endswith(":gate-fixer")`) is unpinned, because the fixture has no gate-fixer session, so a broken match still yields the asserted 0. Evidence: `retro_integration_test.go:193` (`| 0 |` column) with a fixture holding no gate-fixer title.
7. LOW, design. `internal/retroengine/retro.go:780-787` and `:706-707`. No row pins the slug tie-break in `sortByLanding`, and none pins the named-run lookback (`Now - NamedLookback`), which surfaces only in the refusal text. Evidence: `retro_test.go:908-915`, where every landing time in the fixture is distinct; the row at `retro_test.go:966` asserts neither the date nor the lookback.

Tool calls: none.
