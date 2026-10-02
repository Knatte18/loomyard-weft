---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md Decisions 4 and 3
    summary: Decision 4 claims a PassOnCap entry fires at most Attempts re-prompts per run, but Decision 3's reset-on-pass lets an entry that passed and later fails start a fresh streak
  - id: F2
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md Decision 6
    summary: Passed is false only when a non-PassOnCap entry was failing, so a final arrival that stops at a failing PassOnCap entry reports Passed true with a later check entry never run
  - id: F3
    severity: MEDIUM
    location: _lyx/discussion/decision-record.md Acceptance criteria, first bullet
    summary: Existing tests cannot pass with only construction syntax changed - shuttleengine gate tests rely on the removed 0-means-3 default and shedrecipe tests cover the removed gate/gate_attempts keys
  - id: F4
    severity: LOW
    location: _lyx/discussion/decision-record.md Constraints, Auto-mode assumptions; support-log.md ledger item 12
    summary: Refusal-spec reachability is left open for Plan though recipes are embedded, so new shedrecipe construction errors are wiring guards the record can settle now
  - id: F5
    severity: LOW
    location: _lyx/discussion/decision-record.md Scope, Out, attach paragraph
    summary: The attach paragraph prescribes parent-review's closure design; only the per-run count limit and its doc-comment statement are this task's constraint
  - id: F6
    severity: NIT
    location: _lyx/discussion/decision-record.md Decision 1
    summary: The nil-test rewrite sentence is call-site-level instruction already implied by empty-or-nil means ungated
---

### [MEDIUM] PassOnCap bound in Decision 4 is false under reset-on-pass

**Location:** `decision-record.md`, Decision 4 ("A let-through does not reset the count, so a `PassOnCap` entry fires at most `Attempts` re-prompts per run"), read against Decision 3.

**Issue:** Decision 3 resets an entry's count to zero whenever it passes, and the board's ordering rule re-runs every earlier entry at every arrival.
Take `[review (pass_on_cap, attempts 1), validator (attempts 3)]`: review fails, re-prompt, count 1; next arrival review passes, count resets to 0, validator fails, re-prompt; next arrival review runs again (it passed, so it is not let through), fails, count 0 < 1, so it re-prompts a second time.
The "at most `Attempts` per run" bound only holds once the entry is capped; a pass reopens it.
The rationale's "an expensive gate should not run when its verdict would be ignored" is also only true after capping: a passed `PassOnCap` entry re-runs at every later arrival.
The Goal's "one-time request" wording is the board's own and stays; it holds per failure streak, and a request that must not repeat after it passed relies on its closure being idempotent, the same mechanism the Out paragraph already names for attach.
A plan writer taking Decision 4 literally would write a doc comment or test asserting a bound the code does not keep, and `parent-review` would build on it.

**Fix:** Restate the bound: once capped, a `PassOnCap` entry never runs again on this run; a pass resets its count per Decision 3, so an entry that passes and later fails starts a fresh streak of up to `Attempts` re-prompts.
State that a request that must not repeat after it passed, or across an attach, makes its closure idempotent.

### [MEDIUM] Passed at the final arrival ignores check entries not reached

**Location:** `decision-record.md`, Decision 6 (`Passed` is false only when an entry without `PassOnCap` was failing at the final arrival).

**Issue:** The final arrival is `finalize`'s evaluation, which is well defined: it reads the memo Wait stored (exhausted, or re-prompt send failed) or evaluates the list fresh when the memo was cleared by a re-prompt (deadline or session lost between arrivals), exactly as single-gate runs do today (`wait.go` gated-Done branch, `evaluateGate`, `finalize`).
The gap is what that arrival reports when it stops early.
With `[review (pass_on_cap, attempts 2), validator]`, a deadline or failed re-prompt send while review is failing with budget left stops evaluation at review, the validator is "not reached", and Decision 6 yields `Passed` true: the run hands off an artifact the check gate never ran on.
Single-gate rows are unaffected (one non-cap entry, identical to today), so the acceptance claim holds, but the new multi-gate semantics are wrong as written.
Decision 2 names `finalize` as an arrival but does not say that arrival never re-prompts and never changes a count; the per-gate states then follow from Decision 2's ordering rule.

**Fix:** Define `Passed` as true only when every entry without `PassOnCap` that is not off passed at the final arrival, so a failed or not-reached check entry makes it false.
State in Decision 2 that `finalize`'s arrival evaluates the list under the same ordering and cap rules but never re-prompts or changes a count.
Add an acceptance scenario for the not-reached case.

### [MEDIUM] "Only the spec-construction syntax changed" is not achievable

**Location:** `decision-record.md`, Acceptance criteria, first bullet, against Decision 5.

**Issue:** Most `internal/shuttleengine/gate_test.go` tests build `GateSpec{Gate: gate}` with no `Attempts` and depend on today's `Attempts <= 0` means 3 (for instance `TestGate_SendFailsMidLoop_EndsLoopWithAttemptsSoFar`, `TestGate_DeadlineExpiresBetweenAttempts`, `TestGate_FailsOnceThenPassesOnNextTurn`).
Once 0 means off, a mechanical migration that keeps `Attempts` absent makes the gate off and those tests fail; they need an explicit budget of 3.
`internal/shedrecipe`'s `entries_*_test.go` gate-config tests exercise `gate`/`gate_attempts` themselves (for example "`gate_attempts` with no `gate`" in `entries_burler_test.go`), a rule that no longer exists, so they are replaced, not re-syntaxed.
As written, the criterion is false the moment the plan follows Decision 5 and 8, and a PR gate judging against it would reject correct work.

**Fix:** Reword: existing behaviour tests keep their assertions, with spec construction migrated to the list form and an explicit `Attempts` of 3 wherever a test relied on the removed default;
shedrecipe's tests of the removed keys are replaced by the `gates` tests listed below.

### [LOW] Refusal-spec reachability is settleable now

**Location:** `decision-record.md` Constraints (Refusal Way-Forward bullet) and the fifth Auto-mode assumption; `support-log.md` ledger item 12.

**Issue:** The record leaves it to Plan to decide whether the new shedrecipe construction errors are reachable through a bound verb.
Recipes are embedded (`contracts/recipes/recipes.go` `//go:embed`, consumed only through `recipes.LoomRecipe`/`recipes.BattenRecipe`), so no on-disk state a run can produce reaches these errors.
That is the refusal spec's wiring-guard class, covered by each section's grouped "wiring guards" row, with no per-row test.
The plan writer can act on the conditional as written, but an open question with a one-line answer costs Plan a lookup and leaves the acceptance criteria silent on refusal work.

**Fix:** Settle it: new shedrecipe construction errors are wiring guards covered by the existing grouped rows, so no refusal-spec rows are added.
Update the Constraints bullet, the Auto-mode assumption and ledger item 12.

### [LOW] Attach paragraph carries parent-review's design

**Location:** `decision-record.md`, Scope, Out, second bullet's continuation.

**Issue:** Completeness-before-leanness: the constraint this task's plan writer needs is that counts are per shuttle run, do not survive an attach, and that the `PassOnCap` doc comment says so, since that is where `parent-review` will read it.
"For the review gate: an already-recorded verdict file passes" is design for `parent-review`'s closure, not anything this task builds; the hub review entry in `support-log.md` already records why it was added.

**Fix:** Keep the per-run limit and add that the `PassOnCap` doc comment states it;
drop the review-gate parenthetical from the record, leaving the idempotent-closure rule general (it also covers F1's pass-then-fail case).

### [NIT] Nil-test rewrite in Decision 1

**Location:** `decision-record.md`, Decision 1, second sentence.

**Issue:** "every `run.gate.Gate == nil` / `gateSpec.Gate != nil` test becomes a length test" is call-site-level instruction of the kind the writer stencil keeps out of Discussion, and it adds nothing beyond "an empty or nil list means ungated".
The hop list in the first sentence mirrors the board entry's own wording and stays.

**Fix:** Drop the nil-test clause.
Decision 9's and the Notes' function names are not defects: they name entry points, and the Notes section is an optional head start.
