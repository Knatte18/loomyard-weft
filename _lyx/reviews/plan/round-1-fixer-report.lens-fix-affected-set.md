```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set: I reran the search behind each set-premise finding (F2, F15, F19) to check the fix covers every member, not only the ones the review named.

Unchecked list: I didn't rerun F15's or F19's searches myself. Both rest on the fixer's own greps already in context: tests asserting the left-pair and fresh-start texts are only `startguard_test.go` and `predrive_test.go`, and `ResumeInWindow` appears only in `adoptcli/doc.go` and `beamcli/doc.go`, with no hit in `docs/`, `pattern/` or `PATTERN.md`. The fix covers every member of both sets. For card 28, every `windowFlag`/`openTerminalFlag` site in context (`arm.go:296`, `arm_seed_test.go`, `lifecycle_integration_test.go:1314`, `wire_seams_test.go:256`, `wire.go:1361`) falls under a target the card now lists.

Findings:

1. **MEDIUM** — card 13 (`InnerRunDeps` gains `Sleeps`; F2's fix to the Edit list) and card 16 (`InnerRunDeps` gains `ReviveAPIError`).
   - **Summary:** F2's set (forced edits outside the card's package) still misses `internal/testkit/envkit`. Its seam inventory names every `InnerRun.*` func seam, and `TestFullEnv_FillsEverySeam` asserts `NilSeams(FullEnv(t))` is empty. A new func field on `InnerRunDeps` therefore needs envkit to fill or list it, but neither card 13 nor card 16 lists `internal/testkit/envkit/envkit.go`. Only card 17 names envkit, for its own change.
   - **Evidence:**
     - `internal/testkit/envkit/envkit.go:65-101` lists `"InnerRun.Sleep"`, `"InnerRun.Awake"`, `"InnerRun.Pause"` and the other seams.
     - `envkit.go:370` builds `InnerRun: battenshed.InnerRunDeps{…}`.
     - `internal/testkit/envkit/envkit_test.go:16-19` runs `TestFullEnv_FillsEverySeam`.
     - `grep -ln envkit _lyx/plan/*.md` returns only `17-land-drops-ready.md`.
   - **Uncertain:** I didn't confirm whether the map at lines 65-101 is the nil-legal list or the filled list. Either way a new seam has to be added to one of them.
   - **Fix:** add `internal/testkit/envkit/envkit.go` (FullEnv's `InnerRun` literal, or the seam map) to cards 13 and 16. Card 31 is the card that fills these seams in production.

2. **LOW** — card 13's Edit list for F2.
   - **Summary:** `internal/shedrecipe/recipe.go:306` embeds `InnerRun battenshed.InnerRunDeps`, and `shedrecipe/entries_batten.go` sets `InnerRunDeps` fields. Card 13 only adds fields, so I found no forced edit there.

Tool calls:
1. Grepped battencli tests for `InnerRun.Sleep` and listed `InnerRunDeps` users outside battenshed, to rerun F2's search for card 13 beyond the four named steps; context didn't have it.
2. Read the `InnerRunDeps` uses in `shedrecipe`, `envkit` and `run_test.go`, plus `wire_seams_test.go:145-160`, to see which of them a new field breaks; context didn't have them.
3. Read the `envkit.go` map header, `TestFullEnv_FillsEverySeam`, and the plan mentions of `Sleeps` and `ReviveAPIError`, to confirm the seam-fill test and which cards add seams; context didn't have them.
