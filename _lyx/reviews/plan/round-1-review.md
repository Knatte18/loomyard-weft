---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/20-drop-getter-delegation-tests.md
    summary: Card 20 deletes the Runner LoadSkill, SkillUnknown and CompactedSince delegation tests on the claim that other tests cover the behavior, but none of the cited tests reach the Runner methods
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/17-orch-role-run-loop.md
    summary: Card 17 rewrites three sections and orders a check of the rest against the recipes, but states no outcome for the Escalations section and its claims pin only the three sections' facts
  - id: F3
    severity: LOW
    class: design
    location: _lyx/plan/21-skill-skip-tables.md
    summary: The two watcher skip tests have different scripts and assertion shapes, so the folded table needs per-row step functions and saves nothing; the card does not say how a row is expressed
---

### [MEDIUM] Card 20's coverage citation does not hold

**Location:** `_lyx/plan/20-drop-getter-delegation-tests.md`, second bullet.

**Issue:** `findings.md` F2 says to delete only where no existing test covers the behavior.
The card deletes `TestRunner_CompactedSince_Delegates` and `TestRunner_LoadSkillAndSkillUnknown_DelegateToCapability` and cites three coverers.
None of the cited coverers calls the `Runner` methods.
The `claudeengine` tests (`TestSkillLoadSequence`, `TestSkillUnknown`, `TestCompactedSince_*` in `usage_test.go`) call `(&Claude{})` directly.
`TestStartup_*` goes through `run.go`'s own `loader.SkillUnknown`, not `Runner.LoadSkill`.
The orch watcher tests use a fake session; the real `orchcli/session.go` adapter is the only caller of the `Runner` methods, and no test reaches it.
What remains for `Runner.LoadSkill` and `Runner.SkillUnknown` after the deletion is only the missing-capability refusal.
The strand guards (`FindRun`, `requireLiveStrand`) and the typing of the load sequence into the pane are no longer exercised.
`Runner.CompactedSince` is a pure pass-through; `TestRunner_SessionMethods_ErrorOnPlainEngine` already covers its refusal, so its delegation test is safe to delete.

**Fix:** Keep the `CompactedSince` deletion.
For `LoadSkill` and `SkillUnknown`, replace the delegation test with one test of the guard behavior (an unknown guid and a dead strand both refuse before anything is typed), or fold the one assertion that the load sequence is typed into the plain-engine refusal test as a second case.
State in the card which test covers the behavior, by name, as the fix.

### [MEDIUM] Card 17 leaves F1's last sentence unsettled

**Location:** `_lyx/plan/17-orch-role-run-loop.md`, intent opening and claims paragraph.

**Issue:** F1 asks to check the rest of the stencil against `batten-recipe.yaml` and `loom-recipe.yaml`.
The card names the three misstated sections, says "check every other section ... the same way", and lists no result.
A writer cannot tell which other sections need a change.
The check turns up at least one candidate.
"Escalations from a child" tells the orch to record a decision and then message the driver to resume, while `internal/loomcli/circling.go`'s header says the circling verbs resume nothing and the operator runs `lyx loom start` afterwards, and `internal/battenshed/innerrun.go` resumes a halted child only on a recorded decision.
The stencil also names `lyx loom decision`, whose verb is `lyx loom decision add`.
The claims paragraph pins facts for the run loop, PR-Gate and landing only, so a regression in any other section the check changes is unpinned.

**Fix:** State in the card the outcome of the check for each remaining section, including the Escalations resume path (who resumes after `circling continue` or `accept`, `decision add`) and the exact decision verb, and add a claim for each fact the card changes.

### [LOW] Card 21's watcher table is underspecified

**Location:** `_lyx/plan/21-skill-skip-tables.md`, second bullet.

**Issue:** The startup pair differs only in spec, `endsTurn` and `unknown` inputs and an elapsed assertion, so a table fits.
The watcher pair does not: `TestWatcher_UnknownSkillSkippedAndLogged` sets `skillUnknown`, ticks twice and ends two turns, while `TestWatcher_SkillLoadTimeoutSkips` advances a fake clock around the timeout and asserts no skip before it.
A shared table body needs per-row step closures, which is a harder-to-read test than the two it replaces.
The card says each row keeps its assertions but not how the differing steps are expressed.

**Fix:** Either state the row shape (a `setup` field for `skillUnknown`, a `before`/`after` clock step, and the expected ticks), or leave the watcher pair as two tests and fold only the startup pair.

## Notes on the other focus questions

- Card 18: `internal/shedrun` and `internal/loomshed` do not import `contracts/stencils`, so the claims row can build the park-marker claim from `shedrun.ParkMarkerFileName` without a cycle.
  The card's token boundary matches the deleted test's.
  No defect.
- Card 19: the claims row for the role stencil follows card 17 in commit order, so the dependency holds.
  Folded checks keep what the tests asserted.
  No defect.
- `## verify:` runs `go test ./...` and `go test -tags integration ./...`, which cover every package cards 17 to 21 target.
  No defect.
- Fidelity: cards 17 to 21 are each licensed by F1 or F2.
  The sweep leaves the per-module `*_SkillsAndParentDirective` tests unchanged; each pins one module's wiring at its own surface, so keeping them is defensible and is not recorded as a finding.
