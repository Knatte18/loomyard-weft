---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/testkit/envkit/envkit.go:43
    summary: "Cards 13 and 14 add Env and deps seams that three NilSeams audits reject, and no card lists envkit nilLegal, battencli intentionallyNil or FullEnv's Darn fill"
    fix: "In card 13 add `internal/testkit/envkit#nilLegal` to Edit and state in its Intent that nilLegal gains the six new Env paths DocumentSubject, CodeSubject, MergeSubject, MergeInOnly, WeftHead and RejectReview, whose nil is the documented default of a Bouncer row that names no subject, so batten's wiring and loom's wiring before card 15 keep passing NilSeams. In card 14 add `internal/testkit/envkit#nilLegal` (path Darn.RecordRejection, whose nil records nothing) and `internal/battencli#intentionallyNil` (a row Rework.KeepReviewRecords with the reason loom-only seam, batten has no rework segment) to Edit, and state both in its Intent. This was checked with lyx loom validate-plan --overlay together with the other fixes of this review, and the combined run reports nothing the baseline does not."
  - id: F2
    severity: LOW
    class: scope
    location: internal/shedengine/goto.go:38
    summary: Goto's doc comment keeps saying the goto entry ends the target segment's bounce episode, which D1 makes false, and no card names it
    fix: "In card 2 add `internal/shedengine#Goto` to Edit and add a sentence to its Intent that Goto's doc comment stops saying its OutcomeGoto entry ends the target segment's bounce episode. Cards 3 and 4 also edit Goto, and a later card may list a member an earlier card lists. This was checked with lyx loom validate-plan --overlay together with the other fixes of this review."
  - id: F3
    severity: LOW
    class: scope
    location: internal/loomcli/doc.go:149
    summary: The arbiter paragraph of loomcli's doc.go lists the preconditions that make a resume unsound, and card 16 adds a guard refusal to them without targeting doc.go
    fix: "In card 16 add `internal/loomcli/doc.go` to Edit and add to its Intent that the arbiter paragraph's list of unsound-resume preconditions gains a goto target the told goto guard refuses, and that the paragraph on the tackler and arbiter re-launch names the hand-back with the refusal text. This was checked with lyx loom validate-plan --overlay together with the other fixes of this review."
  - id: F4
    severity: LOW
    class: scope
    location: internal/loomcli/cli.go:381
    summary: Card 5 adds pause_reason to the status envelope core but rewrites only the three pause help texts, while the loom and batten status help texts enumerate the envelope's keys
    fix: "In card 5 extend the Intent sentence on the three pause help texts so that the status help texts in `loomVerbTexts`, `shedVerbTexts` and `battenVerbTexts`, which are already Edit targets, also name the pause_reason key among the keys the envelope carries; no test pins the prose. This was checked with lyx loom validate-plan --overlay together with the other fixes of this review."
  - id: F5
    severity: LOW
    class: scope
    location: internal/loomcli/bootstrap.go:148
    summary: Card 18 rewrites the awaitRunLockHalted comment that names a reflection after a blocked halt, but that comment sits in the doc of dispositionForHandshake, which is not a target
    fix: "In card 18 add `internal/loomcli#dispositionForHandshake` to Edit, beside `internal/loomcli#awaitRunLock`, so the paragraph that says a blocked halt runs the friction reflection after Run released the lock is covered by a listed member. This was checked with lyx loom validate-plan --overlay together with the other fixes of this review."
---

### [BLOCKING] Seams added by cards 13 and 14 break the NilSeams audits, and no card lists the sites

**Location:** `internal/testkit/envkit/envkit.go:43`, `internal/battencli/wire_seams_test.go:12`, `internal/loomcli/wiring_seams_test.go:12`
**Issue:** This is the method gap the focus directive names: a card adds a field to a struct, and a test or table that audits the struct whole is unlisted.
`envkit.NilSeams` reports every nil func field of a `shedrecipe.Env`, and three tests assert on its answer.
`TestFullEnv_FillsEverySeam` requires `NilSeams(FullEnv(t))` to be empty.
`TestWire_EverySeamFilled` in `internal/battencli` and in `internal/loomcli` each require every nil path to sit in a local `intentionallyNil` table, or in envkit's `nilLegal`, and require every table entry to be nil.
Card 13 adds six `Env` seams: `DocumentSubject`, `CodeSubject`, `MergeSubject`, `MergeInOnly`, `WeftHead` and `RejectReview`.
It lists `FullEnv`, which then fills them, but batten's wiring never fills a loom-only seam, so batten's test reports all six.
Loom's own wiring fills them only in card 15, so loom's test reports them from card 13 on, and the card gate over card 13's impacted packages would fail.
Card 14 adds `PRReworkDeps.KeepReviewRecords`, a required seam that loom's `wire` fills and batten's does not, so batten's test reports `Rework.KeepReviewRecords`, and batten's table already lists every other `Rework.*` seam.
Card 14 also adds `DarnDeps.RecordRejection`, which `FullEnv` builds at `envkit.go:326` without a fill, so `TestFullEnv_FillsEverySeam` reports `Darn.RecordRejection`;
a loom seed never fills it, so loom's test reports it too, and batten's.
The six Env seams and `Darn.RecordRejection` have a documented nil default (a Bouncer row naming no subject, a darn rework that records no rejection), which is what `nilLegal` holds, and the recipe build refuses a missing seam once card 15 names a `subject`.
`Rework.KeepReviewRecords` is required, so batten lists it the way it lists its other `Rework.*` seams.
The earlier fixes list each test that builds a struct whole by literal and each doc that enumerates its fields, and this gap is the same method failing on a site those fixes did not name: an audit that walks the struct by reflection.
A card that adds a func field to `Env`, `PRReworkDeps` or `DarnDeps` lists `envkit`'s `nilLegal` and each `intentionallyNil` table the field lands in, or the filled `FullEnv` and wiring.
The fix is checked against the plan gate, together with F2 to F5.

### [LOW] Goto's doc comment still says a goto ends the bounce episode

**Location:** `internal/shedengine/goto.go:38`
**Issue:** `Goto`'s doc says the appended `OutcomeGoto` entry "ends the target segment's bounce episode".
D1 removes exactly that rule, and card 2's Intent rewrites the comments of `gotoEndsEpisode`, `OutcomeGoto`, `OutcomeReroute`, `episodeStuckCount` and `doc.go`, but not this one.
Card 2 does not target `Goto`, and cards 3 and 4 target it for other changes, so nothing states that the sentence changes.

### [LOW] Card 16 changes the arbiter's observable preconditions without touching loomcli's doc.go

**Location:** `internal/loomcli/doc.go:149`
**Issue:** The `# Arbiter` paragraph lists when a resume reads `hand-on` with cause `precondition`, ending "or a goto target the run's current row does not admit".
Card 16 makes the arbiter reject a verdict naming a row the settled-segment guard refuses and hand back with the refusal text when its re-launch is refused, and the decision record's Scope requires the `doc.go` of every package whose observable behaviour changes.
Card 15 lists `internal/loomcli/doc.go` for the subject wiring, but card 16 is the card that builds the guard and does not.

### [LOW] The status help texts omit the new pause_reason key

**Location:** `internal/loomcli/cli.go:381`
**Issue:** The loom status help text (`loomVerbTexts.Status.Long`) and the batten one list the keys of the envelope: producer, state, error, pause flag, activity, history length, slug and parent, interrupt policy, `paused_since` and `paused_by`.
Card 5 adds `pause_reason` to the generic core, and D4 says `lyx loom status` and `lyx shed status` show the reason.
Card 5's Intent rewrites the three pause help texts and `doc.go` but never the status texts, although the same three glyphs are targets.

### [LOW] An edit named in card 18's Intent falls in a symbol the card does not target

**Location:** `internal/loomcli/bootstrap.go:148`
**Issue:** Card 18 says "`awaitRunLock`'s doc ... and the `awaitRunLockHalted` comment beside it are rewritten".
`awaitRunLock`'s doc is a target.
The paragraph that says "after a blocked halt `lyx loom run` runs the friction reflection after `shedengine.Run` has already returned" is the doc of `dispositionForHandshake`; the `awaitRunLockHalted` constant's own comment names only post-run bookkeeping.
`dispositionForHandshake` is not in the card's Edit list.

## Rejected

- The `GenerationStart` mark on Shed's budget arm (card 2) answers the attack-surface question.
  Its Intent names the one producer that may set it (the Bouncer's successful seed of a new generation, Merge-Review's included), how often (once per generation, never on a re-bounce or a degraded seed), what it skips (the budget comparison and `budget_capped`, even over a spent episode) and what bounds it (a generation starts only when the subject changed, or with no settle record since the last judge spawn).
  The mark never folds into a neighbouring entry, since the seed Stuck is counted and `foldsInto` takes only `BudgetExempt` Stucks.
- Card 18 carries the `PATTERN-friction-capture` change in the same commit as the code: `PATTERN.md` and `pattern/PATTERN-friction-capture.md` are Edit targets, and the Intent gives the entry text, which matches the declared text and keeps the current entry's `(test)` tag and background link.
- Struct-whole sites of the other field-adding cards are listed or unaffected.
  Card 3's `TestStatus_JSONRoundTrip`, card 5's `TestStatusCmd_EnvelopeKeySet` and `TestStatusCmd_FoundEnvelope_GenericCore`, card 7's `templateConfig` helpers, `wantLocalKeys` and the two `doc.go` key lists, card 15's `wantSegments` and `wantSegment`, card 17's `TestNewShed_FieldCompleteAssembly` and card 19's `TestWait_LogsSessionStateBesideItsClassification` are all targets.
  The `isZeroResult` checks of the shuttle attach and startup tests read a `Result` no `finalize` fills, so they are unaffected.
