---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/20-drop-getter-delegation-tests.md
    summary: Card 20 deletes TestRunner_CompactedSince_Delegates on the claim that claudeengine's CompactedSince tests cover it, but those tests call the engine directly and the kept test covers only the refusal
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/20-drop-getter-delegation-tests.md
    summary: The replacement Runner test asserts a live-strand result for LoadSkill only, so Runner.SkillUnknown's live path (capture plus the engine's detection) loses its only test
  - id: F3
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/21-skill-skip-tables.md
    summary: The watcher table needs a third differing step (ending the two turns on the unknown row) and two different post-skip assertions, which the card's own two-field limit forbids and its row shape does not state
  - id: F4
    severity: LOW
    class: design
    location: _lyx/plan/21-skill-skip-tables.md
    summary: The startup fold states no row shape although the two folded tests differ in spec, scripted reactions and elapsed assertion
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/plan/20-drop-getter-delegation-tests.md
    summary: fakes_test.go is an Edit target with no stated change, and the ImpactSummary claims fake state is removed from the Claude engine, which holds none
  - id: F6
    severity: MEDIUM
    class: design
    location: _lyx/plan/17-orch-role-run-loop.md
    summary: The After landing outcome is left to the writer for the read the friction notes and the driver's report step, and no claim pins the result
  - id: F7
    severity: LOW
    class: scope
    location: _lyx/plan/17-orch-role-run-loop.md
    summary: Several facts the card changes have no pinned claim, so a stale phrase can return unnoticed
---

### [MEDIUM] Card 20: the CompactedSince coverage claim does not hold

**Location:** `_lyx/plan/20-drop-getter-delegation-tests.md`, bullet on `TestRunner_CompactedSince_Delegates`.
**Issue:** The card says the engine's `CompactedSince` tests in `claudeengine` cover the behavior.
Those tests (`internal/shuttleengine/claudeengine/usage_test.go`) call `c.CompactedSince` on the `Claude` type, never `Runner.CompactedSince`.
`TestRunner_SessionMethods_ErrorOnPlainEngine` covers only the refusal on an engine without the capability.
After the deletion nothing exercises the Runner's success path: the `toldErr` guard, the capability lookup and the argument pass-through.
The sibling `TestRunner_ContextTokens_Delegates` is the same kind of test and stays, with no reason given.
**Fix:** Reword the bullet to say what is true: the engine's tests cover the detection, the refusal test covers the missing capability, and the Runner's pass-through keeps no test of its own by choice.
Say why `TestRunner_ContextTokens_Delegates` stays, or fold it into the same decision.

### [MEDIUM] Card 20: the replacement test drops Runner.SkillUnknown's live path

**Location:** `_lyx/plan/20-drop-getter-delegation-tests.md`, bullet on `TestRunner_LoadSkillAndSkillUnknown_DelegateToCapability`.
**Issue:** The card argues the startup and watcher tests never call `Runner.LoadSkill` or `Runner.SkillUnknown`, which is correct: `Run.loadSkill` in `internal/shuttleengine/run.go` calls the reed and the loader directly, and the watcher tests use a fake session.
The replacement covers refusals for both methods but a live-strand result only for `LoadSkill`.
The deleted test's `SkillUnknown` assertions (a matching capture answers true, another skill answers false) are not replaced, so the method's live path (capture plus the engine's detection) is untested.
**Fix:** Add to the replacement a live-strand `SkillUnknown` case that answers true for the skill the capture names and false for another.

### [MEDIUM] Card 21: the watcher table contradicts its own limit

**Location:** `_lyx/plan/21-skill-skip-tables.md`, second bullet.
**Issue:** The card allows a row data plus two optional steps, `unknown` and `beforeTimeout`, and says that if a row needs more, the pair stays as two tests.
The unknown row also needs a third step, ending the two turns (`e.endTurn("t2")`, `e.endTurn("resumed")`) before its final assertion, which the card itself says.
The two deleted tests also end differently: the unknown test calls `e.assertReload("clear", LastHandoff)` after those turns, while the timeout test asserts only that the calls after the clear are two long with `skill:ly:board` second.
"Assert the calls after the clear" does not say which assertion each row keeps.
**Fix:** Either name a third optional step (such as `endTurns`) and the per-row final assertion in the row shape, or keep the watcher pair as two tests under the card's own escape hatch and fold only the startup pair.

### [LOW] Card 21: the startup fold states no row shape

**Location:** `_lyx/plan/21-skill-skip-tables.md`, first bullet.
**Issue:** `TestStartup_SkillWithoutTurnEndSkippedAtTimeout` and `TestStartup_UnknownSkillSkippedAtOnce` differ in the spec (a 50ms `SkillLoadTimeout` against the default), the `endsTurn` and `unknown` lists passed to `skillStartFixture`, the expected typed sequence, and the elapsed assertion (at least the timeout against exactly zero).
The card says each row keeps its sequence and assertion but never names the fields, so a writer must invent the shape, including how one table holds two kinds of elapsed check.
**Fix:** State the row fields: the spec, the `endsTurn` list, the `unknown` list, the want-typed string, and the minimum and maximum elapsed (a zero maximum for the unknown row).

### [LOW] Card 20: a target with no stated change and an inaccurate impact line

**Location:** `_lyx/plan/20-drop-getter-delegation-tests.md`, Edit list and `ImpactSummary`.
**Issue:** `internal/shuttleengine/fakes_test.go` is an Edit target, but the card changes nothing in it: `cyclerEngine` is declared in `internal/shuttleengine/session_test.go`, and `skillFakeEngine` stays.
The `ImpactSummary` says the unused fake state is removed "from shuttle and its Claude engine", yet `claudeengine/session_test.go` holds no fake state; it loses only `TestDefaultSkillLoadTimeout`.
**Fix:** Drop `fakes_test.go` from the Edit list, or state its change, and correct the `ImpactSummary` to name only the shuttle fake.

### [MEDIUM] Card 17: the After landing outcome is left to the writer

**Location:** `_lyx/plan/17-orch-role-run-loop.md`, the "After landing" bullet.
**Issue:** The current section reads "pull, read the friction notes and the driver's report, and amend the board entries".
The card says what remains is pulling in the prime and amending the board entries, and to "drop any step that reads from the removed task worktree".
It never says whether the friction-notes-and-report step goes, stays, or moves, so the writer must decide where those files live.
The card otherwise states each remaining section's outcome (Escalations changes, the other seven change nothing), which is what the focus asks for; this bullet is the exception.
**Fix:** State the outcome for that step in the bullet, and add a claim that pins it.

### [LOW] Card 17: changed facts without a pinned claim

**Location:** `_lyx/plan/17-orch-role-run-loop.md`, the claims paragraph.
**Issue:** The claims cover the pair creation and teardown, the resume after a decision, Finalize closing the pull request, `lyx loom start` after circling, `lyx loom decision add`, and the three mustNot phrases.
These changed facts have no claim: the removed "start the run again so it finalizes" step, the `lyx loom reject <review-file>` form, and the sync with main, board-done marking and push in Finalize.
**Fix:** Add a mustNot claim for "start the run again" and must claims for the reject form and for Finalize marking the board task done and pushing.
