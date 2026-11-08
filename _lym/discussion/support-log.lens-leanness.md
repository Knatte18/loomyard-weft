```yaml
lens: leanness
outcome: reported
```

Task: a leanness review of the decision record and support log, looking for content to move or cut.

1. **MEDIUM — D1, registration sites.** D1 lists every registration site for `arras`: `internal/shedrun/seed.go`, `internal/loomrecipe/names.go`, `internal/shedcli/table.go` and `internal/battencli/wire.go`, plus the reversed `seed_test` case.
   - That is a call-site enumeration, which Step 2 fences off as plan-level fact, and `relevant.md` already carries those files.
   - The decision needs only "registered wherever `motif` and `raddle` are". Move the file list to `## Notes for the plan writer` or drop it.
2. **MEDIUM — D2 Rationale, a rejected alternative.** "where re-entering Raddle-Write would start a fresh Opus session that rereads the board and the code" argues against a rejected alternative.
   - The spec's compaction rule says Decisions carry Decision and Rationale only, with rejected alternatives in the support log, which already lists this one.
   - Fix: keep the positive reason (the arbiter serves the Bouncer ring, and a cold fixer reads only the proposal, the record and the findings). Cut the comparison.
3. **MEDIUM — Scope Out bullet vs D5, and the third Open risk.** Scope says "Raddle's choice of each part's shape, recipe and profile is unchanged", but D5 limits an arras run to two outcomes and refuses `beam` and multi-part `tasks`.
   - The third Open risk ("moves raddle's shape freedom into two outcomes…") is the trade-off D5 chose, not an unknown.
   - Fix: narrow the Scope bullet to "unchanged for the `raddle` recipe; D5 limits the shapes an arras run may file". Drop the Open risk, since D5's rationale already states why.
4. **LOW — The Auto-mode assumptions repeat decisions.** All four bullets restate D2, D8, D5 and D6 almost word for word, and the support log's Question ledger repeats them again.
   - Say once: in the record, cut each bullet to the assumption itself plus a decision id, such as "the fixer is cold, not Raddle-Write re-entered (D2)", without restating the mechanism.
5. **LOW — The Sketch point is stated twice.** D1's last sentence ("An arras run never makes its entry a Sketch, since the recipe holds a `RaddleFile` row") repeats the first Notes bullet about `finishesAsSketch`. Keep one; the Notes bullet with the symbol is the better home.
6. **LOW — The `ClearMotifRun` note is stated twice.** That Notes bullet repeats D6's last sentence and the `relevant.md` line for `internal/battencli/cleanup.go`. It adds only "tests the recipe name", which could join D6's sentence.
7. **LOW — Two Scope Out bullets.** "No `cartoon` recipe" and "No change to the hub's `default_recipe`" rule out nothing a plan writer would reach for.
   - The `cartoon` point is glossary history that D9's glossary update covers.
   - Cut both, or keep only `default_recipe` if D9's "default for a tapestry-sized note" could be misread as a config change. That misreading is plausible, so keeping that one bullet is defensible.
8. **LOW — The D3 Rationale cites history.** The `slack-v2` incident appears again in D7's Rationale. One mention, in D7 where the mechanism lives, is enough; D3's Rationale can say "an Added after Discussion entry carries no decision id, so the mechanical checks cannot see it".

What I checked and kept: D6's body-composition detail (keeping the former body is a design choice, not plan detail), the Constraints' PATTERN list (required), and the Goal (short, and no restatement worth cutting).

I made no tool calls; the context held everything this lens needed.
