# Fixer report — plan, round 1

Every finding's premise was checked against the code and held; all fifteen are fixed.
`lyx loom validate-plan` reports `ok: true` after the fix, with informational `create-new-unit` findings only.

## Changes

- **F1** (card 27), fixed as proposed: dropped the `tracedGlyph` Create target; a traced glyph is kept when non-empty and holding no whitespace, control character or backtick (`strings.ContainsFunc` over `unicode.IsSpace`/`unicode.IsControl`, `strings.ContainsRune`); added that this is a prompt-safety character check, not a glyph grammar (PATTERN-glyph-conversion-chokepoint), and that `tracedLocation` reads a `path:line`.
- **F2** (card 11), fixed as proposed: Edit group with `internal/loomshed#loomshedAllowedImports`, an Intent clause adding `gopkg.in/yaml.v3` with its comment, and an `ImpactSummary`.
- **F3** (card 25), fixed as proposed: Edit group gains `cmd/lyx#treeSitterAllowedLinks`; the Intent names the new entry and the guard; the `ImpactSummary` names the list.
- **F4** (card 4), fixed as proposed: `patternPlanFindings` re-signed to take `cards`; `planContextFindings` passes its `cards` argument; `pattern-removes-prescribed` reads only cards not recorded done, while the projection and `pattern-line-unprescribed` keep every live card; a `TestNewPlanGate` row with the fencing card recorded done expects no `pattern-removes-prescribed`.
  The proposal's re-sign example is used as written.
- **F5** (cards 11, 12, 14), fixed as proposed: card 12 states the start's source is whole-second and the recorded start floored; cards 11 and 14 state that `WrittenAt` keeps full precision, so a same-second write reads as after the start and the floor costs at most an extra round; test rows use whole-second starts, and card 11's `AmendmentSettledBy` rows add a write later in the start's own second.
- **F6** (card 4), fixed as proposed: the way forward names a pending card that edits `PATTERN.md`, or a card inserted with `lyx loom insert-card` after the last begun batch; the `TestNewPlanGate` row asserts that wording.
- **F7** (card 13), fixed as proposed: the darn refusal sits right after `deps.darn()` answers true and before `checkDarnRecord`; the reaching-input line says the darn row's fake is a darn run whose record check is never reached.
- **F8** (card 5), fixed as proposed: a `TestPatternBaseReader` row whose merge base answers the empty string expects `OverviewAt`'s refusal, and the Intent names that reaching input.
- **F9** (card 11), fixed differently: the restore-from-HEAD alternative is dropped and the way forward names the operator, `lyx fabric commit` and the re-run of the refused step (the goto, `lyx loom resume` for the commit seam, or the `decision add` call).
  The proposed "operator removes the file" was not kept as the only step: the fix-proposal self-check showed that a removed record admits no goto (card 14 refuses "this target with no record"), so the goto re-run could never progress.
  The way forward instead rewrites the file as a valid record (the pending titles, the current UTC `written_at`), or removes it when no amendment is pending; the test row asserts the reworded text.
- **F10** (card 12), fixed as proposed: an unparsable round start reads as no known round start, logged at Warn and returning the zero time and false; the error return carries only a failed read of the settle record; the test row reads false.
- **F11** (card 13), fixed as proposed, with its mechanism made explicit: new `errPlanUnparsed` and `decisionWayPlanUnparsed` Create targets; `planApprovedReader` stats the overview first (an absent one answers false), returns a `ParsePlan` error holding an `*fs.PathError` as a read failure, and wraps every other `ParsePlan` error in `errPlanUnparsed`; the verb refuses that with `decisionWayPlanUnparsed`, naming `lyx loom validate-plan` and recording without `--amends-plan`; `TestDecisionVerb_AmendsPlan` gains a verb row and rows over `planApprovedReader` itself.
- **F12** (cards 6 and 10), fixed as proposed and widened: card 6's Edit group gains `checkCardTypeMissing`, `CardTypeCustom`, `tools/sandbox/SANDBOX-WEBSTER-SUITE.md`, and, from the self-check, `checkCustomNotAlone` and `planparser_test#goodPlanDir`; the Intent names each count and the spec's "other six" `Custom` rule, and puts `parse.go`'s package and const-block comments in the `parse.go` edit; card 10 also changes the stencil's and the rubric's "multi-label combination of the other six".
- **F13** (card 8), fixed as proposed: Edit group gains `internal/planparser/doc.go`; the Intent has it state `CardRefKind`, `DependencyEdge` and the pair rule.
- **F14** (card 13), fixed as proposed: an absent decision record is not refused at the pre-read; the append's `fs.ErrNotExist` case refuses with `decisionWayNoRecord` after restoring the amendment record, the decision record staying absent; a test row pins it.
- **F15** (cards 18 and 19), fixed as proposed: `AmendmentTarget(recipe string)` answers for loom, darn and batten and refuses other names as `Rows` does; `TestAmendmentTarget` covers all four cases; card 19 calls `loomrecipe.AmendmentTarget(c.recipe)`.

## Self-check

- fix-consistency, LOW: card 13's darn refusal lost its way forward when split from the unapproved-plan refusal. Fixed as proposed: the darn refusal names recording without the flag, and each refusal states its own way forward.
- fix-proposal, MEDIUM: F9's remove-and-re-run fails on the goto path, since a removed record admits no goto. Fixed with own change (see F9).
- fix-proposal, LOW: `errPlanUnparsed` named no rule for telling a parse failure from a read failure, and an absent overview does not wrap `fs.ErrNotExist`. Fixed as proposed: the stat-first and `*fs.PathError` rule (see F11).
- fix-affected-set, LOW: further seven/six label counts (`validate.go` `checkCustomNotAlone`, `parse_test.go` `goodPlanDir`, the plan stencil's and rubric's "other six", the spec's `Custom` rule). Fixed as proposed: cards 6 and 10.
- fix-claims, MEDIUM: on card 13's absent-decision-record path, the append-error restore would write an empty decision record. Fixed as proposed: the restore writes prior bytes only for a record that existed, and the test row asserts the record stays absent.

## Disputed

None.

## Deferred

None.
