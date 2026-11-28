```yaml
lens: forced-edits
outcome: reported
```

Forced-edits review: what the plan's changes force outside the members its cards list.

1. **BLOCKING** — Card 5 (`constructor` type re-sign). Card 5 adds a seams parameter to the registry's constructor. Code that builds or passes `map[string]constructor` outside the members card 5 lists stops compiling.
   - `internal/shuttleengines#keepWarmWith` and `internal/shuttleengines#KeepWarm` (keepwarm.go:32-41,53) build and pass that map.
   - `internal/shuttleengines#TestKeepWarm` (keepwarm_test.go:71-74) holds constructor literals.
   - `internal/shuttleengines#TestNew` (shuttleengines_test.go:136-164, `forksWithoutAudit`, the `build(tt.cfg, tt.dirs, tt.constructors)` call) holds them too.
   - Card 5 lists none of these, nor `build`, whose parameter type changes; it has `build` under Uses only.
   - Fix: list them under Edit, or keep the constructor signature and wrap the seams outside it.

2. **MEDIUM** — Card 1 (`Renderer.Passages` re-sign). Card 1 lists only `TestShowAndPassagesReadTheShippedLayerFirst`, but `internal/heddleengine#TestRender` also calls `r.Passages` and compares its result (render_test.go:754, 758, 762, 793). After the re-sign to `[]Passage`, that test no longer compiles. Fix: add `internal/heddleengine#TestRender` under Edit.

3. **MEDIUM** — Card 1 (`passageItems` item name). Card 1 renames the item from `heddle passage <n>` to `heddle passage <slug>`. `internal/shuttleengine/codexengine#TestPrepare_AppendsStandingPassages` asserts `"heddle passage 1"` in its oversize refusal (prepare_test.go:415). The card says the two standing-passage tests "keep their assertions on the launch text", so this row would fail. Fix: the Intent should state that this `wantErr` changes to the slug.

4. **LOW** — Card 6/8 (new package and verb). `docs/overview.md:639` lists `lym shuttle run|open|state|await|await-forks` and the shuttle packages, and the package tree at docs/overview.md:324-329 lists the shuttle and seat packages. Neither names `internal/shuttleparity` nor `lym shuttle manifest`, and no card lists `//docs/overview.md`. Fix: card 8 extends `//docs/overview.md`, adding the verb, the package and its tree line.

5. **LOW** — Card 8 (cli.go header). The Intent says "`cli.go`'s header comment names the verb", but the file comment lies outside `internal/shuttlecli#Command`'s span, and the card's only cli.go ref is `Command` under Extends. Fix: name the file self glyph's header change, or drop the claim.

6. **LOW** — Card 5 (`PATTERN-shuttle-provider-seam` background). pattern/PATTERN-shuttle-provider-seam.md says the Codex engine takes the source, hook runner and probe as `codexengine.Seams`, wired in `internal/shuttleengines`. `NewWithSeams` adds a second wiring entry point with caller-given seams, and no card updates that background. Fix: card 5 or card 9 adds a line naming `NewWithSeams`.

7. **LOW** — Card 7 (Uses). Card 7 lists `internal/shuttleengine#promptEngineViolations` under Uses and says the scan walks files "as `promptEngineViolations` does". That function is in a `_test.go` file of package `shuttleengine`, so `internal/shuttleparity` cannot call or reuse it. The Uses entry implies a dependency that cannot exist. Fix: drop the Uses entry and state that the walk is reimplemented over `scankit`.

All 25 of the plan's existing glyphs I resolved with `lym quarry` resolve as found.

Calls made:
- grep for callers of `Passages`, `skillListing` and `passageItems`, the `heddle passage` assertions, constructor use, importers of shuttlecli, target-table asserts and `fakeEngine` (my context had not looked for re-sign callers).
- awk/grep for the enclosing test of render_test.go:754 and keepwarm.go's constructor use, plus a docs grep for shuttle verb lists (my context lacked the function spans and the docs).
- `lym quarry resolve` on the cards' existing glyphs plus a grep of docs/overview.md (my context had not confirmed the glyph spellings or the overview's verb list).
- grep for a test guarding docs/overview.md and for callers of `keepWarmWith` (to tell whether finding 4 is a gate failure and to finish finding 1).
