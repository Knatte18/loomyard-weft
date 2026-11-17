```yaml
lens: forced-edits
outcome: reported
```

Lens: forced-edits. For each change the cards make, I looked for the callers, tests, docs and registrations it forces that no card lists.

**Findings**

1. **BLOCKING**, Card 12 (wiring): `shedrecipe.Env` gains two closure seams, `ReviewTreeID` and `ReviewTreeChanges`, but only `loomCLI.wire` is listed as filling them. Three existing reflective tests require every `Env` seam to be filled:
   - `TestWire_EverySeamFilled` in `internal/loomcli/wiring_seams_test.go:166` (marker `//lym:seam internal/shedrecipe#Env`).
   - `TestWire_EverySeamFilled` in `internal/battencli/wire_seams_test.go:238`. Its builder is the `shedrecipe.Env{` literal at `internal/battencli/wire.go:1100`.
   - `TestFullEnv_FillsEverySeam` in `internal/testkit/envkit/envkit_test.go:50`. It checks `envkit.FullEnv` at `internal/testkit/envkit/envkit.go:394`.

   Neither the batten wire function nor `envkit.FullEnv` is on any card, so the batten and envkit tests fail once the fields exist. Fix: list the batten wire function and `internal/testkit/envkit#FullEnv` under `**Edit:**` on Card 12 and fill both closures there. List the three seam tests too: under `**Uses:**`, or under `**Edit:**` where one changes. `TestWire_DarnRecipe` (`wiring_darn_test.go:67`) and `TestWire_SamplerRecipe` (`:432`) carry the same marker and need listing as well.

2. **MEDIUM**, Card 5 (review-round-checks): Card 5 lists `internal/burlerengine/doc.go` under `**Extends:**`. But `doc.go:557` says "`verdict` and `findings` as the frontmatter's only top-level keys", and Card 5 admits `prior` as a third key. That makes the sentence wrong, so the card has to rewrite it, not only add to the file. Fix: move `internal/burlerengine/doc.go` under `**Edit:**` on Card 5 and correct the line.

3. **MEDIUM**, Card 8 (validate-review-round-checks): `internal/burlerengine/doc.go:560-561` says `lym burler validate-review` "runs CheckReviewFile read-only and needs no hub, mode or git repository". Card 8 changes that behavior (round checks, worktree resolution through `lymcwd`) but edits only `internal/burlercli/doc.go` and the help text. The acceptance criterion that the verb "no longer says the verb needs no git repository" is therefore left unmet in burlerengine's doc. Fix: add `internal/burlerengine/doc.go` under `**Edit:**` on Card 8, or fold the correction into Card 5's edit.

4. **LOW**, Card 2 (fixer-report-dispositions): Card 2 lists `internal/burlerengine/doc.go` under `**Extends:**`. Its skipped-fix sentence (`doc.go:147`, "Fix writes a normal fixer report saying nothing was fixed") stays true but no longer describes the new `spawned: false` frontmatter. Combined with finding 2, one card should hold `doc.go` under `**Edit:**` and Card 2's paragraph should state the Go-written frontmatter where that sentence sits.

No other forced edits found. Every glyph I checked resolves, and every enumeration of causes or round files is covered by a card.

Calls I made:
- `lym quarry resolve` on the 26 doubtful glyphs the cards name: the pre-read held only quarry indexes for some packages. All were found.
- grep for `//lym:seam` on `Profile`, `Finding`, `BouncerConfig`, `BurlerHalfDeps`, `Env` and loom `Config`, and for tests pinning the reworded texts and the parent notice: the pre-read had no seam-marker inventory.
- Read of the seam tests around their markers: needed to see what they assert.
- grep for consumers of the escalation causes and for enumerations of round-file names: the pre-read had only the stencil agent's prose list.
- grep of the doc.go files and stencils for the review keys, the fixer-report and the cause texts: needed to check each card's Extends-versus-Edit choice.
- Read of `burlerengine/doc.go:552-562`, a search for burlercli import guards, and the `Env` builders: needed to confirm the doc line and find every `Env` literal.
