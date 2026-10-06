---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: _lyx/plan/00-overview.md (prune-procedure), _lyx/plan/08-shuttleengine-prune.md
    summary: No card or Shared Decision carries the decision that skill-loading tests are left untouched and kept with a testtiming keep naming skills-in-one-turn
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/09-websterengine-untag.md, 11-reedengine-untag.md, 13-loomcli-untag.md, 15-fabricengine-untag.md, 08-shuttleengine-prune.md, 17-measure-after.md
    summary: Card Targets do not cover files the Intent lets the card change (shared kit fakes under internal/testkit, shuttlefake, test-file fixes in the six packages)
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lyx/plan/00-overview.md (untag-procedure), _lyx/plan/15-fabricengine-untag.md
    summary: The PATTERN-hubforge-fixtures constraint is not carried by the untag procedure, and card 15 invites a temp-dir fixture that hand-assembles a hub
  - id: F4
    severity: LOW
    class: scope
    location: _lyx/plan/02-exclusion-out-of-process.md
    summary: Card 2 excludes any os.Args reference, wider than the record's os.Args index 0 rule, and the record's acceptance line names only index 0
  - id: F5
    severity: LOW
    class: decision
    location: _lyx/plan/03-keep-directive.md
    summary: Card 3 leaves open whether any test pins that a kept test still counts among the covering tests, an acceptance criterion
---

### [BLOCKING] Skill-loading decision is carried by no card

**Location:** `_lyx/plan/00-overview.md` (`prune-procedure`, step 3), `_lyx/plan/08-shuttleengine-prune.md`.
**Issue:** The decision record's "Skill-loading tests are left to skills-in-one-turn" entry forbids any edit, fold, merge or deletion of tests naming `Spec.Skills`, `SkillLoadSequence`, `SkillUnknown`, `DefaultSkillLoadTimeout`, `loadSkill` or `loadSkillsThenPrompt` in `internal/shuttleengine` and `internal/shuttleengine/claudeengine`, of the `SkillLoader` fakes they use, and of the loomcli stub driver scripts.
It also requires a redundancy candidate among them to carry `//testtiming:keep` naming `skills-in-one-turn`.
No file under `_lyx/plan/` mentions skills at all (grep for `skill` finds nothing).
The loomcli stub-driver half is carried by cards 13 and 14.
The shuttleengine half is not, and the tests exist: `internal/shuttleengine/startup_test.go` (`TestStartup_SkillsLoadInOrderBeforePrompt`, `TestStartup_SkillsRefusedWithoutSkillLoader` and the table near line 989), `internal/shuttleengine/session_test.go` (`TestRunner_LoadSkillAndSkillUnknown_*`) and the `skillFakeEngine` in `internal/shuttleengine/fakes_test.go`.
Card 8's fold-and-delete steps (and its permission to extend `internal/testkit/shuttlefake`, which also names the skill methods) are exactly where these tests would be folded, merged or deleted, and `prune-procedure` step 3 would make an unkept candidate among them deletable.
Card 8's "do not touch `claudeengine`" covers only the second package.
**Fix:** Add a Shared Decision (for example `skill-loading-tests-untouched`) stating the record's rule, with the keep reason naming `skills-in-one-turn`, and apply it to cards 8 and 14 (and 7, 10, 12 only where their tests name the listed symbols).
Reference it from card 8's Intent next to the `shuttlefake` sentence, and from `prune-procedure` step 3.

### [MEDIUM] Targets do not cover what the Intent lets a card change

**Location:** cards 9, 11, 13, 15, 8 and 17.
**Issue:** `untag-procedure` lets an untag card add a shared fake under `internal/testkit/<kit>/`, yet cards 9, 11, 13 and 15 target only their own package unit.
Card 8's Intent lets it extend `internal/testkit/shuttlefake`, which is not among its Targets.
Card 17's Intent lets it fix test files in the six packages "if the final rerun finds a gap", and none of those files is a Target.
A reader (or the batcher) sees a narrower write scope than the Intent grants.
**Fix:** Add the kit path as a Target to the cards that may touch it, or state in each Intent that a shared fake is created through a separate card.
For card 17, add the six package units as Targets or move the conditional fixes to the package prune cards.

### [MEDIUM] PATTERN-hubforge-fixtures is not carried by the untag procedure

**Location:** `_lyx/plan/00-overview.md` (`untag-procedure`), `_lyx/plan/15-fabricengine-untag.md`.
**Issue:** The record's Constraints say a test that keeps a real hub builds it through `hubforge`, and a moved test drops the hub and never hand-assembles one.
`untag-procedure` says a moved test runs against "a fake or in-memory fixture" and never states the hubforge rule.
Card 15 says a moved test "runs against an in-memory or temp-dir fixture", which reads as permission to assemble a hub by hand in a temp dir.
**Fix:** Add one sentence to `untag-procedure` that a test needing a real hub builds it through `internal/hubforge` and a moved test drops the hub rather than hand-assembling one, and reword card 15's fixture sentence to match.

### [LOW] Card 2's os.Args rule is wider than the record

**Location:** `_lyx/plan/02-exclusion-out-of-process.md`, the first bullet.
**Issue:** D1 and the acceptance criteria name `os.Args[0]`.
The card excludes any reference to `os.Args` because the scan cannot tell the index, which also excludes a test that only reads `os.Args[1:]`.
The wider rule is conservative, but it is scope the record does not license and the card does not mark as a deliberate narrowing of D1.
**Fix:** Either restrict the scan to an index-0 selector expression (the AST distinguishes `os.Args[0]`), or state in the card that the wider rule is a deliberate bound and name it in the doc text the card writes.

### [LOW] Card 3 leaves the kept-test-still-covers assertion open

**Location:** `_lyx/plan/03-keep-directive.md`, the Tests paragraph.
**Issue:** The card says `classify` is unchanged, then plans a `TestClassify` row only "if `classify` reads the keep at all", otherwise "the render row suffices".
The acceptance criterion "a kept test still counts among the covering tests of the others" needs one definite test.
A render row with a kept candidate does not show that the kept test appears in another test's Covering column.
**Fix:** Plan one `TestRenderPackage` or `TestClassify` row in which a kept test appears in another candidate's covering list, and drop the conditional wording.

## Settled without a finding

- Cards 1 and 3's new tests (a pure argument-vector function, the keep scan) and card 4's `splice_test.go` cover behavior no existing test covers: `cmd/testtiming` has `TestClassify`, `TestParseProfile`, `TestRenderPackage`, `TestScanSpawns`, `TestRedundancyTags` and `TestResolveTags`, and none touches `runAlone`'s arguments, the keep scan or a splice.
- Card 2, 3 and 4 Intents state their bounds, and cards 9, 11, 13 and 15 state that production passes today's value, with card 15 naming the destruction chokepoint and write containment.
- The overview `verify:` runs the six packages and `cmd/lyx`, `cmd/testtiming` and `internal/testkit/...` under `integration` and `tmux`, and vets `llm`; per-card `Verify:` blocks add only tag-specific checks the automatic per-card gate does not run, and none is incorrect.
- Card order is sound: cards 1–4 touch only `cmd/testtiming` and docs, so card 5's before measurement precedes every test edit, and card 6 regenerates sections before any prune card.
