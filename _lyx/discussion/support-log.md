# Support log: twb-fta-1

## Interview

Autonomous run: no operator answered; every pick below is an auto-pick recorded in the question ledger.

1. Read the board entry and the related entries `arbiter-fixes`, `glyph-use-audit`, `enclose-kickstart` and `weaver`; `twb-ftb-1` is this entry's bench twin.
   Read issues #505, #509, #517, #522, #511, #500 and #507.
2. Fault 1 at base `800a0e892`: shells no longer expire, but a gated fresh run with outputs present and only unawaited shells finishes `done` at once, and strand removal ends the shells.
   The claude engine's raw-go steer tells every session to run `lyx gate test` as a background Bash call, and the implementer body says `run_in_background` for every gate step.
   The replay corpus case `master-fork-running` shows a fork's leftover shell in Master's background tasks.
   Picked D1 (foreground) as the cause fix and D2 (awaited prefix) as the backstop.
3. Fault 2: Master's stencil already says "Do NOT localize or fix the failure yourself" in its gate-failure section, but shuttle's generic re-prompt says "read it, fix every finding".
   Picked D3, a per-entry re-prompt line.
4. Fault 3: `loadRunWrites` reads Master sessions and their forks only; a recovery strand's session is never recorded, and each recovery spawn builds a fresh batch record.
   Picked D4, run-level recovery-session list in a third write group.
   `recover-batch` returns `output.Ok` for every terminal digest outside the `batch_failed` path; picked D5.
5. Fault 4: the recovery prefix bans `lyx webster` and the shared body bans mutating `lyx fabric`; neither names the gate commands as allowed.
   Picked D6 per the board's wording.
6. Faults 5 and 6: quarry v0.2.1 answers `found` for `internal/testkit/tmuxkit#tmuxAllowed` (base already holds the var-plus-init workaround), for unexported members and for `_test.go` members, and `ambiguous` for `internal/proc#StartTime`.
   `planglyph` accepts only `found` or `multipart` at six sites.
   Picked D7 (one disjoint-ambiguous predicate in `planglyph`) and D8 (detail names the answer).
7. Lens review, one fork per lens, all six returned:
   - premises: #511 `rusageStats` per-GOOS claim false (the per-GOOS pair declares `maxRSSBytes`) — fixed, the cause is now recorded unexplained and D8 carries the diagnosis.
   - premises, goal-scope, scenarios (merged): slot-busy gates wait up to `cli_wait_sec` before exit 75 — fixed in D1's rationale and Open risks.
   - premises: a shell label can fall back to its description — fixed in D2's bound.
   - premises, goal-scope, board-fit, leanness (merged): the foreground steer for the orch and `CLAUDE.md` was unlicensed and blocks a conversational orch (`orch-split`) — fixed, D1 now scopes foreground to Webster's implementer body, the steer only drops "background", and `CLAUDE.md` is out of scope.
   - goal-scope: an unheld gate shell still ends with no "result not read" record — fixed, D2 makes the note and warning say so.
   - goal-scope: D5 left out Master's ladder wording and other callers — fixed in D5.
   - goal-scope: the #522 exit-0 path is not pinned — fixed in part: the criterion now covers stuck and dead terminal digests separately; which one #522 saw is unknown.
   - goal-scope, pattern-fit, scenarios (merged): `_test` suffix wrongly treated as a disjointness constraint — fixed in D7 and an acceptance row.
   - goal-scope: the Plan-time half sat only in assumptions — fixed, D7 states it.
   - leanness: D3's rationale argued a rejected alternative — fixed, cut.
   - leanness: D7 and D8 rationales leaned on evidence and rejected alternatives — fixed, trimmed.
   - leanness: the #511 finding stated three times — fixed, one assumption line remains.
   - leanness: Scope "In" re-listed the decisions — fixed, reduced to a pointer plus docs.
   - leanness: Constraints bullets that bind nothing — fixed, three removed.
   - leanness: test-budget note is padding — rejected: `CLAUDE.md`'s rule binds the plan's `Edit:` list, which the plan writer acts on.
   - pattern-fit: `loom-plan-spec.md` missing from the docs scope — fixed.
   - pattern-fit: refusal-spec rows quoting D7 or D8 texts — fixed in Scope.
   - pattern-fit, scenarios (merged, BLOCKING): a recorded recovery session with no transcript would fail the audit and break later recoveries and resets — fixed, D4 reads it as no evidence, with an acceptance row.
   - pattern-fit: D3's line restates the stencil's procedure — fixed in part, the line now only orders "not yourself" and points at the section; the order itself is the fix.
   - pattern-fit: 3600000 is a Claude-specific number in a shared stencil — rejected: the body already names Claude's `run_in_background` and Agent tool.
   - scenarios: D7 did not fail closed on unreadable files or unparsable constraints — fixed.
   - scenarios: GOOS and GOARCH tags and `unix` were unspecified — fixed in D7, with a `unix`/`windows` acceptance row.
   - scenarios: D5 persistence, sync and re-entrant behavior — fixed, persistence and sync stay as today; a re-entrant call follows the same terminal path.
   - scenarios: a stuck Master now waits on a gate shell — fixed, stated in D2's bound with an acceptance row.
   - scenarios: begin-batch re-resolution and drift detection not named — fixed in D7's site list and an acceptance row.
   - board-fit: `arbiter-fixes` composes with D7's scope-guard union — fixed in Open risks.
   - board-fit: `weaver` could reuse D4's list — rejected: no requirement today.
   - board-fit: the D7 predicate should defer to a future quarry — rejected: hypothetical.
   - board-fit: D3's Go line names a Claude fork — rejected: Webster's procedure is fork-based on every engine it runs on today.

## Rejected alternatives

- Fault 1, stop the shell explicitly and record its result as discarded: lyx cannot stop a shell, since the provider reports no pid, and a discarded result still leaves the gate unread; holding and foregrounding keep the result.
- Fault 1, match awaited shells per shell segment (`&&`, `;`, `|`): parsing shell text for a backstop D1 already makes rare.
- Fault 1, keep background gates and raise `background_shell_wait_min`: the threshold expires nothing at base, so it changes no outcome.
- Fault 1, foreground gates for every lyx-spawned session and `CLAUDE.md`: blocks a conversational orch for up to an hour, and the fault is Webster's.
- Fault 2, a verb that spawns the verify fixer: a verb cannot spawn a fork; it would need a new strand role.
- Fault 2, a parent-write check inside the verify gate closure: duplicates the run-exit audit for a wording fault.
- Fault 3, start the second recovery from a clean tree with the first attempt's diff as reference: discards work, needs a destructive reset outside `fabricengine/destroy.go`'s chokepoint, and contradicts the prompt's "continue from the uncommitted work".
- Fault 3, attribution by batch only: a fork's leftover from any batch is already "this run", so narrowing recoveries alone would be inconsistent.
- Fault 3, add recovery sessions to `recordedSessions`: the run-exit audit would read a recovery's writes as Master parent-writes.
- Fault 6, change quarry to honour build constraints: a quarry release and a version bump for a judgment `planglyph` can make from the files quarry already names.
- Fault 6, accept every `ambiguous` answer: admits two declarations a build compiles together.
- Fault 6, a new Plan-time finding for Create targets that may become build-tagged: Plan cannot know how a Create will be written.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 (auto-pick, revised after lens review): does the foreground rule apply to the orch too? No; it lives in Webster's implementer body, and the shared steer only stops naming background.
- Q2 (auto-pick): which prefix does D2 add? `lyx gate test`, matched by prefix as shuttle does today.
- Q3 (auto-pick): where does the verify rule go? The `verify` entry's re-prompt line, keeping the `Gate findings recorded at` opening.
- Q4 (auto-pick): attribution by batch or by run? By run, in a write group separate from Master's and the forks'.
- Q5 (auto-pick): which terminal digests exit non-zero? Every status but `done`; the running snapshot stays 0.
- Q6 (auto-pick): where does the `tmux`-step rule go? The recovery prefix only, as the board asks.
- Q7 (auto-pick): quarry or `planglyph` for build constraints? `planglyph`, one predicate for every acceptance site.
- Q8 (auto-pick): is #507's delete-and-recreate case in scope? No; with glyph Edit working it is an Edit.
- Q9 (auto-pick): is a disjoint-ambiguous Rename old side accepted? No, it stays refused like a multipart one.
- Q10 (resolved by reading): does quarry resolve unexported and test-file members? Yes. `tmuxAllowed` was per-tag; `rusageStats` is a single declaration in the landed code, so its #511 failure stays unexplained.
