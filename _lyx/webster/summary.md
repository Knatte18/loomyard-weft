# Webster run stopped at batch 05 on plan drift

Four of sixteen batches finished and were recorded `done`; the run then stopped as `stuck`.

- **01-plan-first-card** (`8e5ffcaf5`): `planparser` parses an optional `first_card` overview key, and the numbering check counts from it.
  It also records the key in `contracts/specs/loom-plan-spec.md`.
  Informational scope warnings only: touched symbols outside the card's declared glyphs, all in `internal/planparser` and its tests.
- **02-review-exempt-classifier** (`c8ee03049`): `planparser.ReviewExempt` in `reviewclass.go`, with its tests.
  Informational scope warnings only.
- **03-webster-archive-run-record** (`09bb584cf`): `websterengine.ArchiveRunRecord` and its tests.
- **04-rework-told-values** (`9c3f84bd9`): `loomshed.ReworkTold` and a session factory for `NewPRRework`, wired through `Env.ReworkSpec`.

`begin-batch 05` refused with `plan_drifted`: card 5's member glyph physically overlaps card 9's own file self glyph naming `internal/loomshed/prrework.go`.
The way forward is for an operator to edit the plan so the cards match the tree and run `lyx webster rebaseline --card NN` for each edited card, then re-run `lyx webster run`.
Batches 05 to 16 did not begin, and no integration suite ran.
