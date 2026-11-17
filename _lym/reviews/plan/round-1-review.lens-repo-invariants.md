```yaml
lens: repo-invariants
outcome: reported
```

Lens: repo-invariants. For each card I checked the test seams, the doc text the card falsifies, and card order.

**Per card**
- **Card 1:** seams held (no shared struct changes). The new test is tier1 and fits `taskpatch_integration_test.go` (tier1, package `fabricengine_test`): held. Doc: `fabricengine/doc.go` is under Extends and the card only adds verbs: held. Order: held.
- **Card 2:** seams held (`Profile` and `RunOpts` have no reflective seam test). Doc: the `burlerengine/doc.go` skipped-fix sentence is under Edit: held. The `PATTERN-sole-parsers.md` addition falsifies nothing, though a second "fixer report" section now sits beside landingshed's (not a broken invariant). Order: held.
- **Card 3:** seams held. Doc: `shedadapters/doc.go` is under Extends and the card only adds: held. Order: held.
- **Card 4:** seams held (`BurlerHalfDeps` has no `//lym:seam` marker). Doc: Extends on doc.go: held. Order: uses Card 3's `TargetSnapshot`: held.
- **Card 5:** seams held (`Finding` and `Profile` are unmarked). Doc: the only-keys sentences are under Edit on the `burlerengine/doc.go` and `ParseCurrentReview` doc comments: held. Order: held. Card 6 updates the only-keys rule in `burler-review-format.md` afterwards, which is fine.
- **Card 6:** doc and order held.
- **Card 7:** seams, doc and order held. It uses the Card 2 parser and the Card 4 record, both earlier.
- **Card 8:** test tier held (new tier1 file). Doc: **broken** in `internal/burlercli/validate.go` (F2).
- **Card 9:** doc held. The `renderEscalation` doc comment ("interpolates only the slug and the brief path") is on the card as an Edit member.
- **Card 10:** doc **broken**: `shedadapters/doc.go` (F1) and the `escalation.go` comments (F3). Order: held, it uses Cards 3, 4 and 9.
- **Card 11:** doc held. `arbiterstop/doc.go`, `loomcli/doc.go:577` and `circling.go` lines 43, 73, 344 and 357 are all covered by the listed members.
- **Card 12:** seams held. The `shedrecipe#Env` reflective tests are listed (`wiring_seams_test.go:166`, `wiring_darn_test.go:67`, `wire_seams_test.go`, `envkit_test.go`). `wiring_beam_test.go:52` also carries `//lym:seam internal/shedrecipe#Env`; its `TestWire_BeamRecipe` is listed under Uses: held. Loom `Config` is covered by `TestConfigTemplate_ContainsEveryConfigYAMLTag`: held. Doc: `docs/overview.md` is under Edit: held.

**Findings**

- **F1 — MEDIUM, consistency, `_lym/plan/10-stalemate-escalation.md` (Extends `internal/shedadapters/doc.go`).**
  The package doc says an unconverged round "escalates to the run's parent … for one of three causes. A fourth cause, decision, escalates a round of any verdict" (`internal/shedadapters/doc.go:140-142`). Card 10 adds a fifth cause on that same unconverged path, so these lines become false. Lines 149-158 also describe the in-window settle order without the stalemate check that now runs ahead of it.
  Card 10 lists doc.go only under **Extends**, which allows additions only, and no card rewrites these lines.
  Fix: move `internal/shedadapters/doc.go` under **Edit:** on Card 10 and rewrite the cause count and the settle order.

- **F2 — LOW, consistency, `_lym/plan/08-validate-review-round-checks.md` (Edit list).**
  Two comments in `internal/burlercli/validate.go` sit outside the `validateReviewCmd` glyph that Card 8 edits:
  - The `validateReviewVerb` const comment says the verb "skips hub, mode and engine wiring, since the check reads only the file it is given" (`validate.go:17`).
  - The file header says the verb runs the same parse every review gate runs (`validate.go:2`).
  
  After Card 8 the verb resolves the worktree and reads the run directory, snapshot record, fixer-report and decision record, so both comments become false.
  Fix: add `internal/burlercli#validateReviewVerb` and the file's header (the self glyph `internal/burlercli/validate.go`) under **Edit:** on Card 8.

- **F3 — LOW, consistency, `_lym/plan/10-stalemate-escalation.md` (Edit list).**
  Two comments in `internal/shedadapters/escalation.go` say "four" and Card 10 lists neither:
  - The file header: "settle an escalation of any of the four causes" (`escalation.go:3`).
  - `ErrEscalationMalformed`'s doc: "its cause is not one of the four values" (`escalation.go:37`).
  
  Card 10 rewrites the four-cause text of `EscalationCause`, `valid`, `writeEscalation`, `readEscalation` and `readCirclingFile` only.
  Fix: add `internal/shedadapters#ErrEscalationMalformed` and the `escalation.go` file header under **Edit:** on Card 10.

**Tool calls**
- grep for `//lym:seam` markers on the structs the cards change, plus a search for four-cause wording: the manifest had no seam-marker inventory and no cause enumerations outside `escalation.go`.
- grep for seam markers in shedadapters, burlerengine, loomengine and fabricengine, and for cause lists in docs and specs: needed to know whether `BouncerConfig` or `BurlerHalfDeps` is walked by reflection.
- grep of `shedadapters/doc.go` for escalation and settle-order text, and of specs and stencils for the only-keys rule: the manifest held no `shedadapters/doc.go` lines.
- Read of `internal/shedadapters/doc.go:134-165`: needed the exact cause-count sentences for F1.
- Lookup of the file, tier and package of `TestFabric_ReadTaskPatch`: needed the tier of the file Card 1's tier1 test lands beside.
