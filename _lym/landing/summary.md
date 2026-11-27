# Motif and its raddle run as the one design-first route

Arras is no longer offered, the Sketch a landed motif run leaves is followed up, and the raddle split gets tighter rules.
The route from a motif run to the filed split then waits on the operator only for the split's approval under autonomy.

## Arras no longer offered

- `boardengine` refuses a write that gives an entry the `arras` recipe, naming `"recipe":"motif"` as the way forward
  (`checkOfferedRecipe` in `internal/boardengine/store.go`).
- The orch role, the `ly:board` and `ly:board-policy` skills, `lym board upsert --help`, `docs/overview.md`
  and the weaving glossary no longer offer it.
  Board-policy proposes a `motif` task for a tapestry-sized note.
- The unknown-recipe errors list offered recipes only, through `shedrun.OfferedRecipeNames`.
  The unsupported-recipe error in `internal/battencli/wire.go` drops arras from its own list.
- Config comments in the `profiles.yaml` header and on `loom.split.reviewer_model` no longer name it.
- The recipe's code stays, and a seed naming it is still accepted.

## Sketches followed up

- Dobby posts a landing notice for a landed motif run whatever `beam.land_notice` reads,
  provided it saw the run before the run passed its Land row; a run first seen past Land gets none.
  The line says the entry is now a Sketch, which runs only once promoted, and replaces the plain landed line.
  `RunView.Motif`, read from the run's child-recipe record in `battencli.DobbyRuns`, marks those runs.
  A motif run that ends without landing follows `beam.land_notice` like any other run.
- The orch role puts the Sketch to the operator: promote it to its raddle run now, or say what it waits for.
- Under autonomy, `role-autonomy.md` has the orch promote the Sketch itself,
  with `depends_on` naming each open entry it waits for.
- A raddle split's approval stays the operator's: the orch puts the waiting split to them with the run's attach command.
  Under autonomy it does so as `[operator] split ready <slug>:`, also posted to Slack, and never approves the split.
  The autonomy texts in the config comments, the orch prompt and the heddle glossary name this third exception.

## Raddle split rules

`contracts/stencils/raddle/raddle-template-agent.md` and `contracts/stencils/loom/loom-rubric-raddle-review.md`
take the section "Proposed stencil changes" of `docs/research/raddle/raddle-split-quality.md`:

- a size bound per darn part, tapestry only for one shared interface,
  and, in a split with more than two tapestry parts, a reason for each of them;
- `darn` with profile `light` by default, `solid` only for a data risk, a wide change or a bundle,
  and `docs` for docs or measuring;
- `depends_on` on work in flight or released that overlaps a part;
- parts that land whole, every clause of a decision mapped to a part, and no part for work filed as a note;
- only whole bullets for criteria and risks, later decisions winning over the record,
  and line counts as package sums;
- the project the filed entries land under named in the text around the table.

When no operator is present, the template sends the notes meant for one into a part's `cost_line` or `next_best.line`,
not `contract.takes`, which is copied into the filed entry.

## Design decisions

- An entry that already holds `arras` keeps taking writes, so a run started under it can finish.
- The refusal sits in the store's write validation, so it also meets other store writes,
  including a board commit replayed from a machine whose binary still allows `arras`.
  `UpsertTasksBatch` checks against the entry stored before the batch, because its snapshot already holds the new one.
- Under autonomy the Sketch is promoted at once with `depends_on`, not held until nothing it waits for is open.
  The raddle run starts only once those entries close, and the board shows why it waits.
  A wait that no entry holds gets a note of its own to depend on.
- The run watch's own landed line for a motif run is unchanged; only the landing notice carries the Sketch line.

## To check by hand

- User-visible text still names arras: the `arras` keys that `internal/runprofile/template.yaml` seeds
  into `profiles.yaml`, and the descriptions of the recipe in `docs/overview.md`
  and `docs/reference/weaving-glossary.md`.
  Decide whether any of these still counts as offering it.
- `internal/battencli/wire_test.go` has a mis-spaced assignment (`err =c.env.SeedChild.WriteSeed`)
  that gofmt would rewrite.

## Merge danger

**Door:** two-way
**Blast radius:** hub
