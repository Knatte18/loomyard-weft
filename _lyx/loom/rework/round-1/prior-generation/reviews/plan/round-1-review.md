---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    location: _lyx/plan/02-writer-fork.md:80
    summary: The live half of D3's ended-after-last-file path has no wordingClaims claim of its own, so deleting it from twill-writer-fork.md fails no test, against the card's own rule that each path gets a phrase only it carries.
    fix: In card 2's wordingClaims bullet (02-writer-fork.md line 80), add a must claim on the twill-writer-fork.md row for the live sub-case's own phrase, "while the session is live the gate re-prompts it", next to the existing "evaluated once over the files as they stand" claim for the no-longer-live sub-case, and add that phrase to line 81's list of phrases the implementer keeps verbatim.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/02-writer-fork.md:40
    summary: Card 2 makes writer_fork and relaunch_rules optional markers of twill-directive.md but never updates the stencil's banner, which card 1 leaves saying it declares these required markers, and it says nothing about banners on the two new part stencils.
    fix: In card 2's twill-directive.md bullet (02-writer-fork.md lines 40-41), add that the banner comment lists the required markers and names writer_fork and relaunch_rules as optional, each filled by Directive with one part stencil; in the twill-writer-fork.md and twill-relaunch.md bullets (lines 39 and 42), add that each opens with a host-neutral banner naming its required markers, in the twill-directive.md banner style.
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/02-writer-fork.md:73
    summary: The writer-fork row of TestDirective_RendersEveryValueVerbatim wants the stencil-only phrases "spawn no second fork0" and "skip its step 1", which card 2's own twill-writer-fork.md wordingClaims row already pins, against card 1's rule that stencil-only wording is pinned once, by wordingClaims.
    fix: In card 2's TestDirective_RendersEveryValueVerbatim bullet (02-writer-fork.md line 73), drop "spawn no second fork0" and "skip its step 1" from the writer-fork row's wants; keep the host values, "Spawn one fork, fork0" (needed for the ordering check), "LAST-FILE last" and "every file in FILES-DIR".
---

### [LOW] D3's live ended-after-last-file path has no claim of its own

**Location:** `_lyx/plan/02-writer-fork.md:80`

**Issue:** Line 80 says each D3 path gets "one claim on a phrase only it carries, so deleting any path fails", and pairs "the run counts as finished" and "evaluated once over the files as they stand" with the ended-after-last-file path, "live and no longer live".
In line 49's text for that path, "the run counts as finished" is the shared opening that both sub-cases rely on.
The live sub-case's rule ("while the session is live the gate re-prompts it on a missing lens review or its findings") has no claim.
An implementer who drops that clause still passes every claim, though the focus directive and the decision record's acceptance criteria name the live and no-longer-live cases separately.

### [LOW] The directive banner is not updated for the two optional part markers

**Location:** `_lyx/plan/02-writer-fork.md:40`

**Issue:** Today's banner in `contracts/stencils/twill/twill-directive.md:3-4` reads "It declares these required markers:".
Card 1 (`01-critical-judge.md:77`) has the banner follow its new marker set, which is all required.
Card 2 adds `{{.writer_fork}}` and `{{.relaunch_rules}}`, which `Directive` fills as optional, and lists the new marker set at line 41, but never mentions the banner.
If the implementer only appends the two names, the banner calls two optional markers required.
Card 2 also gives no banner instruction for `twill-writer-fork.md` or `twill-relaunch.md`, though both are read through the same `stencil.FillOptional` path that strips a leading comment.

### [LOW] The writer-fork render row repeats stencil-only wording that wordingClaims pins

**Location:** `_lyx/plan/02-writer-fork.md:73`

**Issue:** Card 1 (`01-critical-judge.md:97`) sets the rule "the stencil-only wording is pinned once, by `wordingClaims`".
Card 2's writer-fork row of `TestDirective_RendersEveryValueVerbatim` wants "spawn no second fork0" and "skip its step 1".
Its own `twill-writer-fork.md` `wordingClaims` row (line 79) claims both phrases verbatim.
Neither phrase carries a host value or shows which part `Directive` chose, so the row adds no coverage.
"Spawn one fork, fork0" stays, since the ordering check after "SPAWN-POINT-SENTENCE" uses it.
In the other row, "died before its Agent call still spawns once" stays too, since its presence and absence show which part rendered.

## Focus settled without a finding

- D3 in card 2: every failure path, the bound and the re-prompt rule have a bullet at line 49.
  Every path but the live sub-case (F1) has a claim whose phrase only that path carries.
  The re-prompt rule is split correctly: card 1 renders the host-neutral half ("if it has not yet made its lens spawn call") at both hosts, and card 2's writer-fork bullet adds "does step d first if it has not" before deferring to it.
- D5 to D9 and the Constraints: card 1 carries D5 and D6, including both hosts' governing input and dispute evidence kinds, the latter matching the Discussion-Write Step 2 bound in `loom-template-discussion.md:54`.
  Card 3 carries D7 to D9 with the record's wording.
  Card 2's "every lens fork's prompt" change to the Plan-Write added line keeps the quoted line D5 fixes, and only stops fork0 from receiving it, which D2's "nothing else" requires.
  No card adds scope the record does not license.
- Card order and shared files: card 2's directive marker set at line 41 is card 1's set minus `format_step`, `self_check_step` and `self_check_command`, which move to the parts, plus the two part markers.
  `TestStencils_TopLevelMarkers`, `wordingClaims` and the twilldirective tests are consistent across the two cards, and card 2 depends on nothing a later card does.
- Refusal reachability in card 2: `stencil.Fill` checks only the markers a template carries (`internal/stencil/stencil.go:292`), so a blank `ExploreStep`, `InputRecord`, `LastFile`, `FilesDir` or `FormatStep` with a writer fork, and a blank `SelfCheckCommand` or `SelfCheckStep` without one, reach the part's fill, since after card 2 the directive no longer carries those markers.
  No upstream check rejects them first.
  The missing-stencil rows and the two "other part removed renders cleanly" rows reach the selection branch.
- Member glyphs and re-sign arrows: no `Directive`, `planLensDirective`, `discussionLensDirective` or `filledValues` signature changes; `Values` is a struct and `planTwillSpawnPoint` a constant.
  `internal/loomengine/plan.go`'s target is its free-standing file header (lines 1-24, above the package clause), and both `template.yaml` targets are non-Go files, so the file paths are correct.
- Verify: `go test ./...` and `go test -tags integration ./...` run every targeted package, and the section's order matches the plan template's.
- Test economy, card 3: the existing `TestConfigTemplate_DecodesAndIsSelfConsistent` asserts no lens's content beyond the emphasis line, so the three sentence assertions pin acceptance criteria no test reaches yet.
  Card 1's host-test wants pin host values where they land, not stencil wording.
