```yaml
lens: fix-consistency
outcome: reported
```

Lens: fix-consistency. I checked whether this round's changes agree with the cards, criteria and cross-references around them.

**1. MEDIUM — card 19's new constants sentence and card 8's `expectRow` duplicate row-name constants that already exist.**
- Changed text: card 19's Intent now says "`Amend` and `Defer` name the rows they admit, the writer rows, Drift-Verify, Drift-Recheck and Drift-Expect, through constants declared once in driftengine beside `expectRow` and `verifyRow`". Card 8's Create adds `const expectRow = "Drift-Expect"`.
- What conflicts:
  - `internal/loomshed/loomshed.go:27,30,51,52` already declares `NameWebster`, `NameDarnWrite`, `NameDriftVerify` and `NameDriftRecheck`.
  - Card 13's Create adds `plan:internal/loomshed#NameDriftExpect` -&gt; `const NameDriftExpect = "Drift-Expect"`.
  - After both cards, "Drift-Expect" is declared twice: as `driftengine.expectRow` and as `loomshed.NameDriftExpect`. Card 19's "declared once in driftengine" asks for a second copy of the writer, verify and recheck names.
  - That is the stale-twin case `PATTERN-named-constants` forbids, the same pattern F3 cited.
- Card 19's Create list also declares none of the constants its sentence requires, so no card owns them.
- Neither package imports the other today (grep of both found no import), so referencing loomshed's names from driftengine looks possible. I did not check the import graph further.

**2. LOW — card 22's new sentence fixes only part of the darn recipe sentence in docs/overview.md.**
- Changed text: card 22 now says the sentence "names Drift-Expect between Drift-Await and Darn-Write and the `drift` gate entry on Darn-Write".
- What stays stale, at `docs/overview.md:534`:
  - "a `Drift-Baseline` row that reads the record, records the regression scope and starts the baseline capture". The same card's drift-paragraph change has Drift-Baseline "record the scope from config with no agent".
  - "a `Drift-Verify` row that compares the writer's code with it" and "`Drift-Recheck` compares again". Card 16 replaces the compare with containment and the judge.
- The drift-paragraph change has a twin in this sentence that was left as it was.

**3. LOW — card 15's `LoomWeftDirs` description no longer matches the entries now wired into it.**
- Changed text: card 15's wiring now passes "the prefix of its timestamped archive siblings, `LoomFrictionDirRel` followed by `-`" as a `LoomWeftDirs` entry.
- What disagrees: the `Deps` bullet in the same card still defines `LoomWeftDirs []string` as "the weft directories loom itself writes during a row". The new entry is a path prefix, not a directory.
- The exemption bullet now says "everything under `LoomWeftDirs` … and every path starting with the archive prefix". That treats the prefix as separate from `LoomWeftDirs`, while the wiring puts it inside them.

**4. LOW — cards 10 and 19 use `expectRow` but do not list it under Uses.**
- Changed text: card 10 step 3 now reads `PastRow(d.Rows, status row, expectRow)`, and card 19 cites `expectRow`.
- What disagrees: both cards list the card-8 symbols they use as `plan:internal/driftengine#...` Uses entries, for example `plan:internal/driftengine#PastRow` and `plan:internal/driftengine#CheckSeal`. `plan:internal/driftengine#expectRow` was added to neither card's Uses.

Tool calls:
- grep of driftengine, loomshed and plan cards 13–29 for row-name literals and constants: my context had no view of loomshed's constants or of other cards' use of `expectRow`.
- grep of import lines in driftengine and loomshed, plus loomshed.go lines 40–56: my context lacked the import direction and the full drift row-name block.
