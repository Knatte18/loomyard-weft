# Motif and its raddle run as the one design-first route

Arras is no longer offered, every Sketch a motif run leaves is followed up, and the raddle split takes the rules the study of past splits proposed.

## Arras hidden

- `boardengine` refuses a write that gives an entry the `arras` recipe, naming `"recipe":"motif"` as the way forward (`checkOfferedRecipe` in `internal/boardengine/store.go`).
  An entry that already holds the recipe keeps taking writes, so a run started under it goes on to its end.
  The check sits in the store's write validation, so `lym board` and the board viewer's edits both meet it.
- The orch role, the `ly:board` and `ly:board-policy` skills and `lym board upsert --help` no longer list arras.
  Board-policy now proposes a `motif` task for a tapestry-sized note.
- The arras recipe's code and its glossary and overview entries stay.

## Every Sketch followed up

- Dobby's landing notice goes for every landed motif run, whatever `beam.land_notice` reads.
  Its line says the entry is now a Sketch, which runs only once promoted, and replaces the plain landed line.
  `RunView.Motif`, read from the run's child-recipe record in `battencli.DobbyRuns`, tells dobby which runs ran motif.
  A motif run that ends without landing follows `beam.land_notice` as any other run does.
- The orch role puts the Sketch to the operator: promote it to its raddle run now, or say what it waits for.
- Under autonomy, `role-autonomy.md` has the orch promote the Sketch itself,
  with `depends_on` naming each open entry it waits for, so the board shows why the raddle run waits.
  The raddle split's approval stays the operator's: the orch notifies `[operator] split ready <slug>:`, also to Slack, and never approves it.

## Raddle split tuned

`contracts/stencils/raddle/raddle-template-agent.md` and `contracts/stencils/loom/loom-rubric-raddle-review.md` take the section "Proposed stencil changes" of `docs/research/raddle/raddle-split-quality.md`:

- a size bound per darn part, tapestry only for one shared interface, and a reason for each tapestry past two;
- `darn` with profile `light` by default, `solid` only for a data risk, a wide change or a bundle, and `docs` for docs or measuring;
- `depends_on` on every run in flight or released task that overlaps a part;
- parts that land whole, every clause of a decision mapped to a part, and no part for work the record files as a note;
- whole bullets only for criteria and risks, the Sketch's board entry read as the later decision, and line counts as package sums;
- the project the filed entries land under named in the text around the table.

The rubric's finding classes name the new judgements as `design`.

## Design decisions

- Under autonomy the orch promotes the Sketch at once with `depends_on`, rather than waiting until nothing it waits for is open.
  The raddle run then starts only once none of those entries is open, which is the same result without the orch having to be woken again.
  A wait on something no entry holds goes in a note of its own, which `depends_on` names.
- The run watch still sends its own landed line for a motif run whose `beam.run_watch` asks for one.
  Only the landing notice carries the Sketch line.

## Tests

- `TestUpsertRefusesTheArrasRecipe` pins the refusal, that nothing is written, and that the motif write it names is taken.
- `TestLandNotices` gains rows for a landed motif run with `land_notice` off, on and never read, one that ends without landing, and one first seen past Land.
- `TestDobbyRuns_ReadsTheLandOutcomeTheLandingCommitTheEndedEntryThePairAndTheMotifRecipe` pins `RunView.Motif`.
- The arras family-filing tests now set the recipe in the store before the entry's run status, since no write can give it.

## Merge danger

**Door:** two-way
**Blast radius:** orch
