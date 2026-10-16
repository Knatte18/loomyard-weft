---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: _lyx/plan/08-loom-per-round-fan-keys.md (Intent, FanList and fanShapeError; refusal-spec paragraph)
    summary: card 8 adds the fanShapeError refusal for a fan key of the wrong shape but gives it no way forward and lands no refusal-spec row for it, against PATTERN-refusal-way-forward and the record's Constraints
    fix: "In card 8's Intent, state that fanShapeError's Error() reads `value at line %d is neither a fan name nor a list of fan names; set the key to a fan name, or to a list of fan names, with \"\" for a solo or single round`, so LoadConfig's key-naming wrap carries a way forward as it does for modelSpecShapeError. In card 8's refusal-spec paragraph, add a new loom config row, `config: fan key not a fan list`, covering `discussion_fan`, `plan_fan`, `discussion_fix_fan` and `plan_fix_fan` holding a mapping or a list with a non-scalar entry, with that way forward. Extend the existing TestLoadConfig_Refuses row whose `discussion_fix_fan` is a mapping so it also asserts the way forward. Targets stay as they are."
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/07-row-fan-lists.md (Intent, bouncerEntry paragraph) and _lyx/plan/06-bouncer-per-round-exclusions.md (Intent, ClusterFans)
    summary: under the template defaults the Bouncer gets the wiring's all-empty list while the review row runs the recipe's cluster-fan, so a recipe fan with cluster_excludes true never gets exclude_lenses, a regression card 7 says it avoids
    fix: "In card 7's bouncerEntry paragraph, state that bouncerEntry passes the row's RowClusterFans list as BouncerConfig.ClusterFans only when the list holds a non-empty entry, the same all-empty-counts-as-absent rule burlerReviewEntry applies, so a row whose fan comes from the recipe's profile keeps the told cluster_excludes bool. Add a TestBouncerEntry_ClusterExcludes subtest where cluster_excludes is true beside a `[\"\"]` RowClusterFans entry and the prompt still asks for exclude_lenses. In card 6's ClusterFans sentence, say the list is the effective per-round fan list of the partner review row, not the raw wiring value."
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/plan/03-twill-fixer-engine.md (Intent, twillEngineRefusal paragraph; Tests, TestEngine_Fix_Outcomes start-error rows)
    summary: card 3 sets the fixer seat's ModelKey to opts.FixKey, so seatengine already wraps the start refusal with ModelKey.Wrap's own way forward, and twillEngineRefusal's <refusal> slot then yields a message with two different way-forward clauses
    fix: "In card 3's twillEngineRefusal paragraph, define <refusal> as the unmet-requirements refusal without any way-forward clause, so the message carries exactly one `way forward:`, D7's step. Two shapes work: (a) leave the fixer seat's ModelKey zero for a twill round and have twillEngineRefusal cut the trailing `; way forward: ...` that ModelKey.Wrap's default case appends, or (b) keep the seat's ModelKey and have twillEngineRefusal append only the fan key, the remove-the-fan alternative and the `\"\"` caveat, with no second `way forward:`. Say which one, update the refusal-spec row wording to match, and drive the TestEngine_Fix_Outcomes start-error rows through a start error shaped as shuttleengine's start-requirements refusal, so seatengine's Wrap runs, and assert `way forward:` appears exactly once."
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/plan/05-recorded-round-fans.md (Intent, BurlerFixProducer.Call paragraph)
    summary: the fix half reads the recorded names and, when they are missing, archives the round's review and fixer report before the live-fixer probe, breaking Call's probe-before-archive rule exactly in the old-binary case the refusal targets
    fix: "In card 5's BurlerFixProducer.Call paragraph, keep the names read before the probe, since a twill fixer's probe needs the recorded FixFan, but defer the archive and refusal for missing names until after the probe: probe under the single-fixer profile, and when it reports a live fixer, resume it as the single fixer it was spawned as (only an old binary's fixer can be live over a record with no names on a RequiresFans row); archive and refuse only when the probe finds no live fixer. Add a TestBurlerFixProducer_Call_RefusesARoundWithNoAcceptedReview row: a flagless record with a live fixer is resumed and nothing is archived."
  - id: F5
    severity: LOW
    class: scope
    location: _lyx/plan/05-recorded-round-fans.md (Intent, fixedExit sentence)
    summary: fixedExit merges fix_lenses and fix_forks only for a twill result, so a round re-reviewed after a gate-failed twill fix and then run with the single fixer keeps the stale twill lens list and fork count beside fix_fan empty
    fix: "In card 5's fixedExit sentence, have fixedExit always set FixLenses and FixForks from the FixResult, nil for a single fixer, so the single fixer clears what an earlier twill fix of the same round wrote. In TestBurlerFixProducer_Call_FinishedFixIsRecorded, add a single-fixer row over a record already holding fix_lenses and fix_forks that asserts both keys are gone. The three keys D2 protects are unaffected."
---

### [BLOCKING] fanShapeError has no way forward and no spec row

**Location:** `_lyx/plan/08-loom-per-round-fan-keys.md`, the `FanList` sentence and the `refusal-spec.md` paragraph.

**Issue:** Card 8 creates `fanShapeError` and makes `LoadConfig` name it by key "as it names a `modelSpecShapeError`".
The model it copies, `modelSpecShapeError.Error()` (`internal/loomengine/config.go:498`), ends in `modelSpecListWayForward`, and the `config: review or fix key not a model-spec list` row in `contracts/specs/refusal-spec.md:253` covers it.
Card 8 states no way forward for the fan-key version.
Its refusal-spec paragraph covers the model-spec row, the unknown-fan and unreadable-burler.yaml rows, the new empty-list row and the cluster-fan engine row, but no row for a fan key of the wrong shape.
Today a mapping-valued `discussion_fan` fails decoding into a `string` with the generic `unmarshal loom config` error, so this is a new loom refusal.
`PATTERN-refusal-way-forward`, and the record's Constraint built on it, require a way forward, a spec row and a test.
The test row exists (the mapping-valued `discussion_fix_fan`); the way forward and the row do not.

### [MEDIUM] The Bouncer's fan list and the review row's effective fan disagree for a recipe fan

**Location:** `_lyx/plan/07-row-fan-lists.md`, the `burlerReviewEntry` and `bouncerEntry` paragraphs; `_lyx/plan/06-bouncer-per-round-exclusions.md`, the `ClusterFans` sentence.

**Issue:** Card 7 counts an all-empty wiring list as absent beside a recipe `cluster-fan`, "so under the template defaults a recipe `cluster-fan` or `fix-fan` still runs".
The partner Bouncer row gets no such rule.
`bouncerEntry` passes the raw `RowClusterFans` list and stops widening `cluster_excludes`.
Card 6 makes a non-nil `ClusterFans` the whole answer, so the told bool is ignored.
Card 7's interim wiring and card 8's final wiring both give the Discussion and Plan Bouncer rows a list even when the fan is `""` (the template's `FanList{""}`).
Take a recipe that sets `cluster-fan: X` on the Discussion review row and `cluster_excludes: true` on its Bouncer, as `TestShippedRecipe_ClusterExcludesAgreesWithPartnerFan` requires.
Round 1 records `fan: X`, `excludesLenses(1)` compares it with `roundFan([""], 2) == ""`, and the judge never asks for `exclude_lenses`.
Today `internal/shedrecipe/entries_bouncer.go:76` honours the recipe key, and the judge asks.
This breaks the overview's claim that every default leaves today's behaviour unchanged, and it breaks D10's intent that the answer follows the fan the next round actually runs.

### [MEDIUM] The twill engine refusal renders two ways forward

**Location:** `_lyx/plan/03-twill-fixer-engine.md`, the `fixTable` sentence (seat `ModelKey` set to `opts.FixKey`) and the `twillEngineRefusal` paragraph.

**Issue:** `seatengine` wraps every start error with `seat.ModelKey.Wrap(err)` (`internal/seatengine/engine.go:283`).
For a start-requirements refusal, `ModelKey.Wrap` (`internal/shuttleengine/requires.go:49-65`) replaces the error with the refusal text plus `; way forward: set <key> in <module>.yaml to an alias whose engine meets them ..., then re-run the verb, or under the role's row re-step the row`.
For a `DefaultEngine` key it appends `set engine: in shuttle.yaml to an engine that meets them, then re-run the verb`.
Card 3 then builds `... cannot run: <refusal>; way forward: <step>` around that error.
The real message therefore carries Wrap's way forward (re-run or re-step, with no fan option and no `""` caveat) and then D7's step (resume, or remove the fan).
D7 asks for "one wording".
`requirementsError` holds only a flattened message, so the plan must say how `<refusal>` loses Wrap's clause.
The card's tests miss this.
`TestEngine_Fix_Outcomes` injects a fake start error that wraps `ErrRequirementsUnmet` directly.
That error is not shuttleengine's `startRequirementsError`, so `Wrap` returns it unchanged, and the rows pass while the production path renders both clauses.

### [MEDIUM] The unrecorded-names refusal archives before the live-fixer probe

**Location:** `_lyx/plan/05-recorded-round-fans.md`, the `BurlerFixProducer.Call` paragraph ("reads the recorded names after the accepted-review check and before the probe ... Without them it archives the round's review, fixer report and acceptance record").

**Issue:** `BurlerFixProducer.Call`'s contract (`internal/shedadapters/burlerfix.go:94`) is "A live fixer is probed before anything is archived, because archiving renames the very file a live fixer is about to write".
The precedent D2 names, the changed-review exit, sits in `errorExit` after the probe.
Card 5's refusal archives before the probe.
The state is reachable through the case the refusal exists for: an old binary spawned round N's fixer, the process stopped while that fixer was still live, reconcile installed this binary, and `lyx loom resume` reaches the fix row over a record without `fans_recorded`.
On a `RequiresFans` row only an old binary's single fixer can be live over such a record, since this binary never spawns a fixer there without the names.
Probing it as the single fixer before taking the refusal is safe.

### [LOW] A single fixer leaves an earlier twill fix's lenses and fork count in the record

**Location:** `_lyx/plan/05-recorded-round-fans.md`, the `fixedExit` sentence.

**Issue:** A gate-failed fix archives the whole round (`fixedExit`), and the round is reviewed again under the same number.
That review's spawn path re-records the picked names, so after a hub edit of `discussion_fix_fan` the second pass can be a single fixer.
`fixedExit` "merges ... neither for a single fixer", so the record keeps the first pass's `fix_lenses` and `fix_forks` beside `fix_fan: ""`.
That contradicts the acceptance criterion "for a single round an empty `fix_fan` with no lenses or fork count", in the record real runs are measured from.
`acceptedExit` avoids the same problem for `lenses` by always assigning `result.Lenses`.

## Settled focus questions

- Fidelity.
  D1 is carried through D2 to D10.
  D2 is carried by cards 5, 7 and 8, with the Bouncer's degrade in card 6.
  D3 by card 8 and card 7's `ReviewKey`.
  D4 by card 2.
  D5, D6 and D8 by card 1.
  D7 by cards 3, 4 and 7.
  D9 by cards 3 and 5.
  D10 by cards 6 and 7.
  Each Constraint has a carrier: stencils in cards 2 and 3, config through `configengine` and the template fallback in cards 1 and 8, reconcile in card 8, and refusal rows in cards 3, 5, 7 and 8, with F1's gap.
  Card 7's edits to `gateQuiescenceFailureMessage`, `gatedRowFanViolations` and `TestNoGatedRowAuthorizesForkSubagents` are licensed.
  D7 makes a gated `BurlerFix` fixer fork, which contradicts that test's stated rule ("a BurlerFix fixer may carry gates because it never forks"), and the record's Notes ask that the audit hold across the gate's re-prompts.
  The Plan-Write lead already forks under its gates (`internal/shedrecipe/entries_planwrite.go:73-83`).
  Not a defect.
- Card order.
  The focus says card 7 creates `RoundFans`, but card 5 creates it.
  Card 6's `recordRoundFans`, `readRecordedFans` and `roundFan` come from card 5, card 7's `RoundFans` from card 5, card 8's `RowModelKeys` from card 7, and card 3's `lensList` from card 2.
  Every `Uses` symbol a card names comes from an earlier card.
  Not a defect.
- Reachability.
  Card 3's fork violation is reached by `TestAuditClusterRound` rows and the `TestEngine_Fix_Outcomes` write-call row.
  Its engine refusal is reached in name only (F3), and the single-fixer start error keeps today's message.
  Card 5's unrecorded-names refusal is reached by the flagless, undecodable and missing-record rows.
  Note that `acceptedExit`'s merge replaces an undecodable record with a flagless one, so on the live path an undecodable record is rare, but not upstream-blocked.
  Card 8's `fanShapeError` is reached by the mapping row through `ConfigOpenMaps`, and the retired `fan_review` by the repo-default-layer row.
- Attack surface.
  Card 3's Intent bounds the forking fixer by the per-fork violation audit, the one-fork-pass rule and the gate's wait on an outstanding fork.
  Card 7 keeps the cluster-fan refusal on every gated row and names the audits as the backstop.
  Not a defect.
- Test economy.
  Card 5's one new top-level test pins the pre-spawn write, which no existing test covers.
  It keeps `internal/shedadapters` under its row in `cmd/lyx/testdata/test-budget.yaml`.
  Every other new case is a row or subtest of an existing test.
  No card asserts a stencil's or lens's static prose: cards 2 and 3 assert the filled lens bullets, the rubric path and a difference from the solo render.
- Edit-card glyphs.
  Cards 3, 5, 7 and 8 list member glyphs for the Go symbols their Intents name.
  Every re-signed member carries the arrow: `validateFix`, `clusterRulesBlock`, `roundProfile`, `writeRoundFacts`, `gatedRowFanViolations` and `resolveSegmentModels`.
  The other edited members keep their signatures.
- Verify coverage.
  Both `verify:` commands run every targeted package, `contracts/specs`, `internal/configreg`, `internal/configcli`, `internal/testkit/envkit`, `internal/testkit/shedfake` and `internal/loomrecipe` included.
  The packages' only other build tags are `llm` and `tmux`, which are not hermetic tiers.
  The section form is admissible.
