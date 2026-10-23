---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/plan/19-arbiter-amendment-target.md:23
    summary: card 19 computes the arbiter's amendment_target once at wiring time, so a drive that starts before plan approval hands every later arbiter spawn the literal none
    fix: |-
      Edit only _lyx/plan/19-arbiter-amendment-target.md; this fix adds targets and was checked with `lyx loom validate-plan --overlay` (the baseline `{}` run and the run with this fix both report only the informational create-new-unit findings of card 25, no blocking finding).
      1. The `**Edit:**` list gains five lines: `internal/loomcli#arbiterDeps`, `internal/loomcli#arbiterCall.settleFresh`, `internal/loomcli#TestArbiterVerb_SpawnPath`, `internal/loomcli#arbiterFixture` and `internal/loomcli#arbiterFixture.deps`.
      2. In the Intent, replace the whole sentence that begins "`arbiterDepsFor` fills `Prompt.AmendmentTarget` from `arbiterAmendmentTarget(loomrecipe.AmendmentTarget(c.recipe), planApprovedReader(location.AnchorPath()))`" with three sentences.
      First: "`arbiterDeps` gains `AmendmentTarget func() string`, read at each spawn and never at wiring time: the Go driver builds its `arbiterDeps` once and holds them for the whole drive, and the plan is approved mid-drive, so a value read at wiring time would stay `none` for every later stop of that drive."
      Second: "`arbiterCall.settleFresh` sets `promptInput.AmendmentTarget` from `c.deps.AmendmentTarget()` beside the other per-attempt inputs it fills after copying `Prompt`."
      Third: "`arbiterDepsFor` reads `recipeTarget, err := loomrecipe.AmendmentTarget(c.recipe)` and returns `err` like its other build errors, a fail-loud wiring guard that takes no reaching row, since only a broken embedded recipe or a recipe name outside the three can raise it; it then sets `AmendmentTarget` to a closure over `arbiterAmendmentTarget(recipeTarget, planApprovedReader(location.AnchorPath()))`, the same approval `--amends-plan`'s refusal reads."
      3. In the Tests paragraph, after the `TestArbiterPrompt` and `TestArbiterAmendmentTarget` lines, add: "`arbiterFixture` gains `amendmentTarget func() string`, and `arbiterFixture.deps` sets `AmendmentTarget` from it, answering `none` when a row sets none. `TestArbiterVerb_SpawnPath` gains a row: a fixture whose `amendmentTarget` answers `none` when the call is built and `Plan-Burler-Review` when it spawns writes a prompt naming `Plan-Burler-Review`."
      4. In the ImpactSummary, replace "`arbiterPrompt` has one caller, the arbiter call, whose input gains one field;" with "`arbiterPrompt` has one caller, the arbiter call's spawn, whose input gains one field; `arbiterDeps` is built by `arbiterDepsFor` for both the verb and the driver, and each spawn now reads the target afresh;".
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/24-planparser-target-holds.md:15
    summary: card 24 says a type holds its members by an owner-chain prefix without saying the chain is read from parsed glyphs, leaving a string-prefix reading of glyph strings open
    fix: |-
      Edit only _lyx/plan/24-planparser-target-holds.md; the fix changes prose, no target list, so no overlay run applies.
      In the Intent's list of when `TargetHolds` is true, replace "or to a type `member` is a member of (an owner-chain prefix, as `pkg#T` holds `pkg#T.M`)" with "or to a type `member` is a member of, decided on the parsed glyphs' unit, owner chain and name as `tools/plandag`'s `columnsOverlap` decides it (`ParseRef` for a plan ref, `ParseMintedCSharpID` for a C# member id quarry minted), never by a string prefix or split of either glyph string (PATTERN-glyph-conversion-chokepoint), as `pkg#T` holds `pkg#T.M`".
---

### [BLOCKING] Card 19's `amendment_target` is read once at driver start and stays `none` after plan approval

**Location:** `_lyx/plan/19-arbiter-amendment-target.md:23`

**Issue:** Card 19 has `arbiterDepsFor` call `arbiterAmendmentTarget(...)` and store the resulting string in `Prompt.AmendmentTarget`, so the approval is read when the deps are built.
The Go driver builds those deps once and keeps them for its whole life: `internal/loomcli/drivewiring.go:79` calls `goDriverDepsFor` once per drive, line 129 there calls `c.arbiterDepsFor()` once, and the `Judge` closure at line 273 passes that same `arbiterDeps` to `newArbiterCall` for every stop.
`sheddriver.Run` drives from the run's current row onward, and Plan-Bouncer approves the plan in the middle of that drive, so a drive started before approval, such as the one `lyx loom start` begins, reads an unapproved plan at wiring time.
Every arbiter spawn later in that drive then gets `none`.
Only a drive started after approval, such as one after a halt and resume, would name `Plan-Burler-Review`.
D8 says the row is filled "once the current generation's plan is approved", and its acceptance criterion says the arbiter prompt for a stop after plan approval names `Plan-Burler-Review`.
The card's design misses both for the usual continuous drive.
The helper `arbiterAmendmentTarget` already takes `approved` as a `func`, but `arbiterDepsFor` calls it immediately, so the on-demand reader buys nothing.
`TestArbiterAmendmentTarget` and the `TestArbiterPrompt` rows pass regardless, because none builds deps before approval and spawns after it.
The prompt's per-attempt inputs are copied and filled at the spawn, `internal/loomcli/arbiter.go:246` (`promptInput := c.deps.Prompt`), so the target belongs there.
The card also writes the `arbiterDepsFor` call as `arbiterAmendmentTarget(loomrecipe.AmendmentTarget(c.recipe), …)`, which passes a two-value return as one argument and does not compile; the fix reads the error into a local first.
`loomrecipe.AmendmentTarget(recipe string) (string, error)` itself is consistent with `Rows` and `RowNames`, which take the recipe name and refuse an unknown one the same way.

### [LOW] Card 24 leaves the owner-chain comparison open to a string reading of glyphs

**Location:** `_lyx/plan/24-planparser-target-holds.md:15`

**Issue:** `TargetHolds` receives `member` as the glyph string quarry answered and a target as `RefMember`'s canonical glyph string, and must tell whether one is a member of the other.
The card names the rule "an owner-chain prefix" without saying how the chain is read.
`PATTERN-glyph-conversion-chokepoint` allows only `glyph.Parse` plus `Glyph.String` as glyph grammar, and `cmd/lyx/constraintchokepoint_test.go` catches only a `"#"`-trimming `TrimSuffix` and a `.Unit` path read, so a `strings.HasPrefix(member, target+".")` would pass that test while still being a local glyph grammar.
The repository already has the compliant reading in `tools/plandag/graph.go` (`columnsOverlap` compares `Unit` and the owner-chain slice of two parsed glyphs), and the card should point the implementer at it.

## Rejected

- Cards 4 and 5, `pattern-removes-prescribed` against `done-card-edited`: not a defect.
  The check reads only cards not recorded done, so it never asks for an edit of a frozen card, and the projection applies `removes` after the prescribed lines, so an entry a done card already built is still dropped.
  Its way forward names dropping the fenced line from a pending card.
  `pattern-line-unprescribed` names a pending card or one inserted with `lyx loom insert-card`.
- Card 4, the `pattern-headroom` way forward names no card: not a defect.
  A line on a done card passed the run-time headroom lint when its batch was built, so the projection past the line comes from lines of pending cards, which "shorten the prescribed lines" can reach.
- Cards 12 and 14, an unparsable `round_started_at`: not a defect.
  `SettleRoundStart` reads it as no known round start, logged at Warn, so the guard admits and the commit seam keeps the record, as D7 states for an unknown start; no refusal arises.
  The amendment record's own decode error carries card 11's way forward, and a failed read of the settle record is a propagated raw I/O error.
- Card 12 records the round start at whole-second precision where D7 says nanosecond: not a defect.
  The source is the review half's `start`, which comes from the shuttle run's `CreatedAt` written with `time.RFC3339` (`internal/shuttleengine/run.go:619`), so no finer value exists, and the card's comparison against the full-precision `WrittenAt` errs toward one extra round, never a lost amendment.
- Card 23's length guard on `EncloseAt` answers has no named reaching row: exempt.
  A real quarry always returns one answer per target, so no on-disk state reaches it, which `refusal-spec.md` classes as a fail-loud guard needing no reaching test.
- Card 27 against `PATTERN-glyph-conversion-chokepoint`: not a defect.
  The glyph check in `checkedPublishFailure` is a character check for prompt safety, `tracedLocation` reads `path:line`, and no card reads a glyph with a local regex or parser.
  Cards 24 and 25 reach glyphs only through `planparser`, with the one wording gap in F2.
- Card 25 matches a member that no longer resolves through `Delta` where D9 says it resolves at the batch's start SHA: not a defect.
  `planglyph.Resolve` takes no revision, and `Delta`'s deleted, renamed and modified symbols carry the before-side file, so the card reaches the same members the decision names.
