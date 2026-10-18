---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/08-push-lock-pending-flags.md:61
    summary: "Card 8 adds flag-bound test rows one case at a time, so each round finds a side or field still unreached; the respawn bounds are not tested as a side-by-field matrix"
    fix: "In card 8, replace the respawn rows of TestCoalesceFlagged_PendingFlags with one table-driven group (same test function, no new test) whose rows are generated over both flag sides and every field the respawn reads. Sides: a holder owing records that finds pending.code, and a holder owing code under a code path that is not the paired worktree that finds pending.records. Rows per side: (a) a valid flag respawns exactly one request carrying only that side's path, its two slugs and WorkDir equal to the board directory even when the stored one differs; (b) a side path that is not the lock directory's own worktree is removed with a Warn naming the path and spawns nothing; (c) a flag that also carries the opposite-side path respawns a request whose opposite-side path is empty; (d) a PauseSlug that is a dotdot element, and one that is a path separator such as a/b, each remove the flag with a Warn naming it and spawn nothing; (e) the same two cases for TeardownSlug; (f) a flag file that does not decode as JSON is removed with a Warn naming it and spawns nothing. Rows (b) to (f) take the valid flag of (a) and change only the one field, so none can pass on another bound. Add one sentence to card 8's Intent beside the slug rule: a flag of a side the holder does not push that does not decode is dropped with a Warn naming the flag, and one of an owed side is only removed, so a half-written flag never makes a holder return an error or loop."
  - id: F2
    severity: LOW
    class: design
    location: _lyx/plan/15-command-engine-replaces-switched-rows.md:89
    summary: "Card 15 names the Env seams of TestCommandEntry's fixtures piecemeal, so a new fixture seam surfaces each round; the call rows still leave Command.LogPath and Command.Dir unset"
    fix: "In card 15's TestCommandEntry bullet, say the test builds every env from one local helper in internal/shedrecipe's tests (envkit cannot be imported there, since envkit imports shedrecipe) that starts from newTestEnv and sets env.Setting to a fakeSetting over the switch and command references, env.Command.Run to a recorder, env.Command.LogPath to a func returning a fixed absolute path under the test's temp directory and env.Command.Dir to an absolute temp directory; the building rows and minimalEntries' Command row use the same helper with ExecutableSetting left nil, and the call rows use a command text with no task-dir placeholder so TaskDir stays unread. Without LogPath and Dir the call reaches a nil func when it runs the command, and the 30 s and 5 s timeout rows panic instead of asserting the timeout handed to Run."
---

### [MEDIUM] Flag-bound rows are added one case at a time

**Location:** `_lyx/plan/08-push-lock-pending-flags.md:58` (the bounds) and `:61` (the test).

**Issue:** The focus question was whether every field a respawned `PushSpawn` takes from a flag is bounded or rebuilt, and whether a named row reaches each bound.
In the prose, every field is: the side path by the paired-worktree check, the opposite-side path rebuilt empty, both slugs by the separator and `..` rule, `WorkDir` rebuilt as the board directory.
The rows reach only part of that.
- The paired-worktree check and the empty opposite-side path are exercised on a `pending.code` flag only; the `pending.records` branch has no row, though it is reachable: a code-only child, itself a respawn, that finds `pending.records` set by a records-only pusher respawns it.
- `PauseSlug` validation has no row, and the one slug row uses a `..` element, so the separator half of the rule is unreached.
- A flag that does not decode has no stated disposition and no row; a concurrent `markPending` write can leave one for a moment, and `takePending` returns an error in its signature.

A prescribed guard branch with no named input that reaches it is a finding under the rubric's reachability bullet.
It is a design finding, not a scope one, because the gap has recurred: the earlier rounds each added the rows their lens found (the `WorkDir` row, then the slug and opposite-side rows), and each addition left another side or field unreached.
The method that keeps missing is writing the bound's rows as a list of cases; a table over both sides and every field, each row changing one field of a valid flag, leaves no cell to find later.
The rows join the existing test, so test economy holds.

### [LOW] TestCommandEntry's fixture seams are named piecemeal

**Location:** `_lyx/plan/15-command-engine-replaces-switched-rows.md:89`.

**Issue:** The focus question was whether the fixtures set the Env seams the build reads.
The build reads `Env.Setting` and `Env.ExecutableSetting` and passes `Env.Command` through, and the card now sets `Env.Setting` through `fakeSetting`, in `minimalEntries` too, where `settingRef` refuses a nil `Setting`.
Each build refusal the card prescribes (missing, non-string and dotless references, `timeout_s` zero and non-int, unknown key) has a named input row and ends in a way forward clause.
The two timeout rows go further and call the built producer, which per card 14 runs the command in `Dir` with `LogPath(row)`.
`newTestEnv` leaves `Command` zero and the card names only `Run`, so the call reaches a nil func.
An implementer meets this at once, which is why it is LOW; the earlier round's fixture finding (a nil `Setting`) was the same kind of omission, so the fix names one helper that owns the fixture instead of one more seam.

## Rejected

- PATTERN.md size and card 10's cut: the arithmetic holds, so the plan names its cut before card 15's growth and needs no open question for `ly:orch`.
  I recomputed the 37 listed rewrites plus the introductory sentence against the current file: 368 bytes saved.
  The three declared lines add 70, 112 and 189 bytes, and 15556 - 368 + 371 = 15559, under `HeadroomBytes` (15564), with card 10 running before card 15.
  The cut changes only wording outside the three declared entries and keeps names, `(test)` markers and background links.
- Card 15's `PATTERN-recipe-setting-lookup` line departs from the declared text by one word (`told`), and I raise no finding against the plan.
  The declared line is 321 characters, one over `internal/pattern`'s `MaxEntryLineChars`, and the plan-writer stencil tells a card to leave an entry line under the cap, which the plan does.
  Here the listed constraint (judge an entry by its declared text) and the stencil conflict, and no wording keeps both; the one way that does is a one-character raise of the cap, which edits a repo guard the decision record does not license.
  The conflict has a consequence for `ly:orch`: Webster-Review's "Declared PATTERN changes landed" check compares `PATTERN.md` with the declared text and may raise a BLOCKING finding for this entry at landing, and no `Pattern-Departure:` trailer excuses it, since the cap is not a PATTERN entry.
  I checked the cap raise as a plan edit with `lyx loom validate-plan --overlay` (adding `internal/pattern#MaxEntryLineChars` to card 15's Edit list): no finding beyond the baseline, so it is available if the operator chooses it.
- Reachability of a nil `Env.Setting` refusal in `commandEntry`: it is a fail-loud wiring guard, which takes no reaching test.
- The unknown-key row in `TestCommandEntry` beside `TestEveryEntry_RejectsUnknownConfigKey`: the conformance test asserts the key name only, and the card's row asserts the new way-forward wrapping, a different behaviour.
- Verify coverage for cards 8 and 15: every targeted package is in the plan's `## verify:` lists, plain and `-tags integration`.
