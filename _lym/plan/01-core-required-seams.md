# Card 1 — core-required-seams

**Create:**
- `internal/fabriccore#RequiredSeam`
- `internal/fabriccore#SeamRunRecordPaths`
- `internal/fabriccore#SeamRunLocks`
- `internal/fabriccore#SeamStatusCodec`
- `internal/fabriccore#FieldSeams`
- `internal/fabriccore#RefuseUnsupplied`
- `internal/testkit/fabrickit#Builder.Seam`
- `internal/fabriccore_test#TestRefuseUnsupplied`

**Edit:**
- `internal/fabriccore#Snapshot`
- `internal/testkit/fabrickit#Builder.Build`
- `internal/testkit/fabrickit_test#TestBuilder`

**Extends:**
- `internal/fabriccore/doc.go`
- `internal/testkit/fabrickit/doc.go`

**Intent:**
D13's required seams get their core half: the snapshot fact that says whether each required seam is supplied, and the one rule that refuses a verb whose run would use a seam that is not.

- `RequiredSeam` names a seam fabric needs to keep today's behaviour.
  The three constants are the run-records paths, the run locks and shed's status codec, the closed set D13 names.
- `Snapshot` gains the field `Seams map[RequiredSeam]bool`, true for a supplied seam.
  `FieldSeams` is the field a phase names to have the reader fill it (card 3).
- `RefuseUnsupplied` returns nil when every seam of `needed` is true in `snapshot.Seams`.
  Otherwise it returns a `*Refusal` whose `Reason` names every missing seam in the order `needed` lists them,
  and whose `WayForward` names the step for the only process that meets it, a test binary (the lym binary installs the seams at start):
  build the hub through `internal/hubforge`, which installs them once per binary, or call `hubforge.InstallRequiredSeams` before the verb (card 3).
  A nil `Seams` map reads every seam as unsupplied.
- `fabrickit.Builder.Seam` sets one entry of `Seams`, and `Build` clones the map like the others, so a later setter never changes a built snapshot.

`TestRefuseUnsupplied` is a tier0 table test in `internal/fabriccore` over snapshots built with `fabrickit`.
Rows: every needed seam supplied refuses nothing; one unsupplied seam names it; two unsupplied seams name both in `needed` order; a nil map refuses every needed seam; no needed seam refuses nothing.
It is the tier0 core row of D13's refusal acceptance criterion; the verb that reaches it is card 9's.
`TestBuilder` gains one row asserting `Seam` is set and that the built map does not change after a later `Seam` call.
The new and widened tier0 tests add under 1 second to one tier0 run.

The doc.go additions name the required-seam fact and its rule, and the builder's new setter.

**ImpactSummary:** Adds one snapshot field, one rule and one builder setter; no existing caller changes, since no verb asks for `FieldSeams` yet.
