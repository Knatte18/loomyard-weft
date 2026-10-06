---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/09-websterengine-untag.md
    summary: Card 9 says webster's integration tests build one hub through hubforge, but none of them import hubforge; they build a scratch git repo with gitkit.CommitFile.
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/00-overview.md (skill-loading-tests-untouched Applies to) and _lyx/plan/17-measure-after.md
    summary: Card 17 may edit and delete tests in shuttleengine and loomcli, but skill-loading-tests-untouched and the stub-helper rule do not apply to it.
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lyx/plan/00-overview.md (untag-procedure, prune-procedure) and cards 9, 11, 13, 15
    summary: The untag procedure lets a card write under internal/testkit, and the prune procedure lets every prune card do so, but no card's Targets name a testkit path.
---

### [MEDIUM] Card 9 premise contradicts the webster tests

**Location:** `_lyx/plan/09-websterengine-untag.md`, second paragraph of `**Intent:**`.
**Issue:** The card says `runlevel_test.go`, `recordbatch_test.go` and `recoverbatch_test.go` hold the integration tests and "many of them build one hub through `hubforge`".
No file in `internal/websterengine` imports `internal/hubforge`.
Their fixtures are scratch git repos built with `gitkit.CommitFile` (`newRecordFixture` in `recordbatch_test.go`, `newScratchRepo` in `beginbatch_test.go`), plus temp dirs for the plan, reports and webster directories.
Combined with the `untag-procedure` rule that a test needing a real hub builds it through `hubforge`, the sentence steers an implementer to rebuild webster's fixtures on `hubforge`.
That is heavier than what exists and not what the decision record asks for.
The family kind is the "seeded worktree" that `scenario-merge` already lists.
**Fix:** Reword the sentence to name the real fixture: many build one scratch git repo and its plan, report and webster directories, and differ only in the steps after it.
Add that a webster fixture is not a hub, so `PATTERN-hubforge-fixtures` does not apply to it.

### [MEDIUM] Card 17 is outside the protection of the skill-loading and stub-helper rules

**Location:** `_lyx/plan/00-overview.md`, `skill-loading-tests-untouched` `**Applies to:**` (cards 7, 8, 10, 12, 13, 14, 16); `_lyx/plan/17-measure-after.md`.
**Issue:** Card 17 targets `internal/shuttleengine#` and `internal/loomcli#`.
It says that when the final rerun lists a candidate outside "Kept" or a "failed" test, the card fixes it in the package's test files.
That fix can fold or delete a skill-loading test or edit a stub driver script, which the decision record forbids.
The card carries neither the `Applies to:` entry nor the "Stub driver helpers stay untouched" sentence that cards 13 and 14 carry.
**Fix:** Add card 17 to the `Applies to:` list of `skill-loading-tests-untouched`.
Add a sentence to card 17 that a candidate or failure inside the protected set is kept with `//testtiming:keep` naming `skills-in-one-turn`, and that `writeStubDriverScript` and `integrationWriteStubDriverScript` keep their names, signatures and bodies.

### [MEDIUM] Testkit writes are licensed by the procedures but absent from every card's Targets

**Location:** `_lyx/plan/00-overview.md` (`untag-procedure`, `prune-procedure`); `_lyx/plan/09-websterengine-untag.md`, `11-reedengine-untag.md`, `13-loomcli-untag.md`, `15-fabricengine-untag.md`.
**Issue:** `untag-procedure` says a fake that another of the six also needs goes under `internal/testkit/<kit>/`, "the one change outside the card's own package an untag card may make".
The card's Targets cannot name it, because the directory does not exist yet.
Existing kits (`shedfake`, `shuttlefake`) are not named either, though an untag card may need to extend one.
`prune-procedure` also exempts "a shared kit under `internal/testkit/` per `untag-procedure`", so cards 7, 10, 12, 14 and 16 are allowed a write that no Target of theirs covers.
A card whose Intent lets it change a path its Targets omit has a write scope that disagrees with its own Targets.
**Fix:** Settle the kit writes in the plan.
For each untag card that needs a shared fake, read the tests and name the kit's files, or the existing kit's self glyph, in that card's `**Create:**` or `**Edit:**` group.
Where no cross-package fake is needed, drop the licence from the card's reading of `untag-procedure` and keep the fake in the package's `_test.go` files (`PATTERN-testkit`).
Restrict the `prune-procedure` exemption to a kit the prune card names in its own Targets, which today is card 8's `internal/testkit/shuttlefake#` alone.

## Points settled with no finding

- **Skill-loading tests:** `skill-loading-tests-untouched` states that the `Spec.Skills`, `SkillLoadSequence`, `SkillUnknown`, `DefaultSkillLoadTimeout`, `loadSkill` and `loadSkillsThenPrompt` tests, the `SkillLoader` fakes and the loomcli stub driver scripts stay untouched, and that a candidate among them is kept with `//testtiming:keep` naming `skills-in-one-turn`.
  `prune-procedure` step 3 and card 8 both reference it.
  The `skillFakeEngine` named in card 8 exists in `fakes_test.go`.
  Card 17 is the one gap, filed as F2.
- **Hub fixtures:** `untag-procedure` tells a test that needs a real hub to build it through `internal/hubforge` and a moved test to drop the hub, never hand-assembling one in a temp dir.
  Card 15 repeats both and keeps the destruction chokepoint and `os.Root` containment intact.
  The only defect is card 9's wrong premise, filed as F1.
- **`os.Args` rule:** card 2 matches only an index expression on `os.Args` with the literal index `0`, so `os.Args[1:]` stays judged.
  That is the decision record's index-0 wording, and card 2 states it as such.
- **Keep directive:** card 3 plans one definite `TestRenderPackage` row, with no conditional wording, in which a candidate's Covering column names a kept test.
  `classify` is unchanged by the card and never sees the directive, so the kept test stays in the covering pool by construction.
