---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: _lyx/plan/12-bouncer-amendment.md:19
    summary: Card 12 creates Bouncer.startGeneration, which card 11 already landed under that name, so Webster refuses begin-batch 12 with create-already-exists
    fix: |-
      In card 12, rename the new method so it no longer collides with the existing private `startGeneration()` that card 11 landed in `internal/shedadapters/bouncer.go` (it writes generation 1's record when the run directory holds none, and `seedCall` and `approveWithoutReview` call it).
      In the `**Create:**` group, change the line to `- `plan:internal/shedadapters#Bouncer.startNextGeneration` -> `func (b *Bouncer) startNextGeneration(ctx context.Context, settled settleRecord, base string) (shedengine.Outcome, shedengine.OutputPointer, error)``.
      In the Intent paragraph that begins "Case 2 (a live record whose subject changed)", change "call `startGeneration`" to "call `startNextGeneration`", and add one clause that the existing `startGeneration()` keeps writing generation 1's record and is not touched.
      No other mention of the name exists in the plan, so the rename is complete after those two edits.
      Checked with `lyx loom validate-plan --overlay`: the baseline `{}` reports exactly this one blocking finding, and the combined overlay of every fix in this review reports none.
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/12-bouncer-amendment.md:3, _lyx/plan/17-budget-way-forward-texts.md:15, _lyx/plan/18-friction-no-per-stop.md:4
    summary: Method gap, the glyph that carries a rewritten sentence is not listed, for the settledEntry and settledGeneration docs, the ClassifyHalt doc and the halt.go file header
    fix: |-
      Apply three target-list edits, and no change to any Intent.
      Card 12, `**Edit:**`: add `- `internal/shedadapters#Bouncer.settledEntry`` and `- `internal/shedadapters#Bouncer.settledGeneration``, and add one sentence to the Intent saying that the docs of both now describe a live record over a changed subject, and case 3 over a changed subject, as starting a new generation instead of clearing, and that `settledEntry` reports a new action for them, so `Call` tells them from case 4's clear, which both return as `entryClear` today.
      Card 17, `**Edit:**`: replace `- `internal/loomengine/anomaly.go`` with `- `internal/loomengine#ClassifyHalt``, the function whose doc comment carries the "a goto way forward after it" sentence the Intent rewrites.
      Card 18, `**Edit:**`: replace `- `internal/loomcli#loomCLI.reflectHalt`` and `- `internal/loomcli#loomCLI.loomAfterStep`` with the single file glyph `- `internal/loomcli/halt.go``, since the Intent rewrites `halt.go`'s file header, which no member span holds, and `redundant-file-target` forbids listing the file beside its members.
      The three edits were checked together with the F1 and F3 fixes in one `lyx loom validate-plan --overlay` run, which reported no blocking finding.
      One instance has no admissible fix and is only reported: `contracts/stencils/rubric_test.go` carries a file header that says a rubric may carry the six listed markers "and nothing else", which card 13 makes false by adding `amendment`; the constraining check is `edit-test-file`, which refuses a test file under `Edit`, so the fixer defers that sentence.
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lyx/plan/13-recipe-bouncer-subject.md:30
    summary: Card 13 adds the Bouncer row key subject but not to contracts/specs/shed-recipe-spec.md, the contract that lists every Bouncer row key and the Env seam each selects
    fix: |-
      In card 13, add a group after the `**Edit:**` group:
      `**Prosa:**`
      `- `contracts/specs/shed-recipe-spec.md``
      and add one sentence to the Intent: the spec's Bouncer-row paragraphs, which describe `commit_seam`, `skip_seam` and `carry_over` as keys that select among the seams `Env` carries, gain `subject`, with its three values `document`, `code` and `merge`, the seams each selects, the construction error for a named seam the `Env` does not carry, and the construction error for an unknown value.
      A `Prosa` group takes only file self glyphs, which this path is, so `prosa-symbol-target` admits it, and the combined `--overlay` run reported no blocking finding.
  - id: F4
    severity: MEDIUM
    class: scope
    location: _lyx/plan/15-loom-subject-wiring.md:58
    summary: The nilLegal entries for RejectReview and Darn.RecordRejection hide a missing fill from NilSeams, and no listed wiring test asserts that wire fills them, so the reject-file criterion is unpinned at the wiring
    fix: |-
      In card 15's test paragraph, add two assertions to tests the card already lists.
      `TestWire_ReviewKeysReachTheEnv` also asserts `c.env.RejectReview` is non-nil after `wire()` of a loom seed.
      `TestWire_DarnRecipe`'s darn-seed subtest also asserts `c.env.RejectReview` and `c.env.Darn.RecordRejection` are non-nil.
      Both are direct field checks, since `envkit.NilSeams` skips these two paths through `nilLegal` and so cannot report them; add one clause to the Intent saying so.
      No target changes, so no check constrains the fix.
---

### [BLOCKING] Card 12 creates a method that already exists

**Location:** `_lyx/plan/12-bouncer-amendment.md:19`

**Issue:** `.lyx/webster/plan-conflict.md` records `create-already-exists/12-bouncer-amendment[blocking]` at begin-batch 12 with cause `tree_moved`, and the baseline `validate-plan --overlay '{}'` reproduces it.
Card 11, now done, landed `func (b *Bouncer) startGeneration() error` at `internal/shedadapters/bouncer.go:547`, which writes generation 1's record, and card 12 plans a different `startGeneration(ctx, settled, base)` that starts the next generation.
Card 12 is not frozen, so the fix lands in the card, and Webster cannot begin batch 12 until it does.

### [MEDIUM] The carrier of a rewritten sentence is not a listed glyph

**Location:** `_lyx/plan/12-bouncer-amendment.md:3`, `_lyx/plan/17-budget-way-forward-texts.md:15`, `_lyx/plan/18-friction-no-per-stop.md:4`

**Issue:** The focus question for this round is whether every sentence a card rewrites sits on a listed glyph.
Three do not, and each came up because the plan matches a rewrite to the symbol its Intent names, never to the symbol whose comment holds the sentence:
- `Bouncer.settledEntry`'s doc says "A settle record that is live, so its subject changed, clears", and `Bouncer.settledGeneration`'s doc lists case 2 as "clears the run directory and re-seeds" and case 3 as a harvest with no changed-subject branch. Card 12 turns a changed subject into a new generation, so both docs become false, and `settledEntry` also returns `entryClear` for case 2, case 3 with a changed subject and case 4 alike, so the card needs a way to tell the first two from the third. Neither symbol is a target of card 12, and card 11, which owned them, is done.
- `ClassifyHalt`'s doc says the persisted reason "carries the exhausted row and a goto way forward after it". Card 17's Intent rewrites exactly that sentence, and the card lists the file `internal/loomengine/anomaly.go`, not the function.
- `internal/loomcli/halt.go`'s file header says "A halt has one path: the halt note, then the reflection" and "An escalation halt writes its note and skips the reflection". Card 18's Intent rewrites that header, and the card lists two members of `halt.go`, which cover neither the header nor a free-standing comment.
The sites the focus names are otherwise covered: `Goto` is a listed target of cards 2, 3 and 4, `dispositionForHandshake` and `awaitRunLock` are listed by card 18, and the status help texts and `Hooks.StatusExtras` are listed by card 5, which is done.
`internal/loomcli/doc.go` holds no status-key enumeration to rewrite.

### [MEDIUM] The recipe spec's Bouncer key list does not gain subject

**Location:** `_lyx/plan/13-recipe-bouncer-subject.md:30`

**Issue:** `contracts/specs/shed-recipe-spec.md` states, one key at a time, that `commit_seam`, `skip_seam`, `carry_over` and `approve_seam` select among the seams the told `Env` carries, that an absent key leaves the seam nil and that a present key naming a seam the `Env` lacks is a construction error.
Card 13 adds a key of that shape, `subject`, with the same `requireSeam` rule, and lists `internal/shedrecipe/doc.go` as its only enumeration, so the contract goes stale on the card that changes it.
This is the enumerating-doc half of the round-3 carry-over finding, for a recipe-row key rather than a config key.

### [MEDIUM] A nil-legal entry blinds the audit that would catch a missing fill

**Location:** `_lyx/plan/15-loom-subject-wiring.md:58`

**Issue:** Cards 13 and 14 add `RejectReview` and `Darn.RecordRejection` to `envkit`'s `nilLegal`, which is right for batten and for loom before card 15, and card 15 then fills both in `wire` and `wireDarn`.
After that, `TestWire_EverySeamFilled` in `internal/loomcli`, and the `darnSeams` list in `TestWire_DarnRecipe`, are blind to either seam being nil, and neither test the card lists asserts them.
The five subject seams are different: a Bouncer row that names a subject refuses construction when its seam is nil, so a wiring test catches those.
A nil `RejectReview` or `RecordRejection` is silent, so a wiring slip loses the reject file from the amendment note, which is the acceptance criterion "After a PR-Gate reject, the re-entered review segment ... seed names the reject's review file".

## Rejected

- Every audit that walks `shedrecipe.Env` whole is covered: `envkit.NilSeams` and its callers `TestFullEnv_FillsEverySeam`, `TestWire_EverySeamFilled` in `internal/loomcli`, the darn check in `internal/loomcli/wiring_darn_test.go` and `TestWire_EverySeamFilled` in `internal/battencli`. Card 13 lists `nilLegal` and `FullEnv` for the six new `Env` seams, card 14 lists `nilLegal`, `envkit.ReworkDeps` and batten's `intentionallyNil` for `Rework.KeepReviewRecords` and `Darn.RecordRejection`, and loom's `intentionallyNil` needs no row since `wire` fills the required seam and `nilLegal` skips the optional one. Both cards precede the card whose wiring would otherwise break them.
- Every caller of a `PRReworkDeps` that reaches `checkSeams` is listed: `reworkFixture.deps`, `envkit.ReworkDeps` and shedrecipe's `reworkTestEnv`; `wiring_rework_integration_test.go` copies `c.env.Rework`, which card 14 fills, and `reworkParityFixture` never calls the producer.
- The whole-type walks over config and records are satisfied: card 7 is done and its template, `Config`, `templateConfig` and `wantLocalKeys` pins landed together, and `TestConfigTemplate_ContainsEveryConfigYAMLTag` holds because the field and the template key arrive in one card. `landingDeps`'s zero-value walk is untouched by any card.
- `shuttleengine.Result.SessionStates` leaves `isZeroResult`, the `Result` fields compared in `run_test.go` and burler's explicit result envelope unaffected, and the one whole-`Result` comparison, `TestWait_LogsSessionStateBesideItsClassification`, is listed.
- Card order, fidelity to the decision record, the attack-surface and refusal-reachability bullets and verify coverage raised nothing beyond the findings above.
- `lens-review.md` in the plan directory is not a card, and nothing in the plan or this review rests on its content.
