---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-correctness
    location: internal/landingshed/fixerrun.go:143
    summary: A fixer run whose later turn hangs or dies after a failing gate is read as fixed, so Merge-In records the merge and Finalize lands over a tree whose landing gate failed
    fix: In fixerRun.run, derive the fixed end from the gate having passed (session.decided holding a Passed result) instead of from session.reached, which judge sets on every closure call. When the run ends with no passed result and no session.end, set fixerEndTimeout if result.Chair.Outcome is shuttleengine.OutcomeTimeout and fixerEndNoReport otherwise, whether or not the closure ran earlier. Add a TestFixerRun_Outcomes row where turn 1 fails the gate with turns left and turn 2 never reaches the closure, and assert the end is timeout or no-report and not fixed.
  - id: F2
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: internal/loomcli/doc.go:260
    summary: loomcli's package doc still documents the retired PR-Gate alignment, the Merge-Bouncer merge subject, PR-Gate rejects and reject-review.md, none of which the code keeps
    fix: In internal/loomcli/doc.go delete the "# PR-Gate alignment" section (lines 260 to 266, PR-Align and alignment_pr_approve), delete the closing two lines about PR-Gate rejects and reject-review.md (lines 284 and 285), drop "Merge-Bouncer `merge`" from the subject list and the "A merge subject hashes..." sentence, and reword "the verify tiers at Publish, darn's Merge-In and loom's Merge-Review bound it" and the line-140 mention of PR-Gate's alignment agent to name only live rows. Reword the same stale phrases at internal/loomcli/subjects.go:139 and internal/loomcli/decision.go:23.
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/loomcli/validate.go:166
    summary: Help text and doc comments still name PR-Rework's own gate as what lyx loom validate-plan --rework mirrors, a gate card 44 deleted
    fix: In internal/loomcli/validate.go lines 3, 159, 166, 177 and 328, replace the PR-Rework gate reference with the check the flag still runs (the format-only checks over the whole new plan and its first_card, as ValidateReworkPlan does). In internal/battenshed/doc.go:33 and internal/battenshed/notice.go:143 replace the example "awaiting at PR-Gate" with a live row. In internal/loomshed/doc.go:52 drop the "PR-Gate rework" clause.
  - id: F4
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/testkit/envkit/envkit.go:284
    summary: The shared env fixture still builds a Merge-Review segment and a publish gate list that no recipe or gate schema has any more
    fix: In internal/testkit/envkit/envkit.go delete the "Merge-Review" entry at line 284 and the "publish" entry of fixedGateLists at line 225, then run the tests that use FullEnv and fixedGateLists to confirm nothing relied on them.
---

### [BLOCKING] A hung later turn reads as fixed

**Location:** `internal/landingshed/fixerrun.go:143`, with `fixerSession.judge` near line 245.
**Issue:** `judge` sets `s.reached = true` on entry, so `reached` means the closure ran, not that the gate passed.
Take a run where turn 1 fails the landing gate with turns left: `judge` charges the next turn and returns non-terminal findings, leaving `decided` and `end` unset.
If turn 2 then hangs, dies or hits `table.Timeout`, the closure never runs again.
The end `switch` skips `session.end != ""`, skips the timeout case because it requires `!session.reached`, and reaches `default`, which sets `fixerEndFixed`.
Merge-In and Finalize then judge only the change's size and kind caps and go on: Merge-In writes the merge record and Finalize lands, with no passing gate over the tree.
The timeout and no-report ends only work when the very first turn fails to return.

### [MEDIUM] loomcli's doc.go documents retired surface

**Location:** `internal/loomcli/doc.go:260`, `internal/loomcli/subjects.go:139`, `internal/loomcli/decision.go:23`.
**Issue:** Cards 41 to 45 delete PR-Align, PR-Gate, Publish, the rejection record and `alignment_pr_approve`, and card 23 retires Merge-Review, yet the package doc still describes them as live behavior.
No code reads `reject-review.md` or a pending rejection any more.
`CLAUDE.md` requires the module doc to change in the same commit as the behavior, and the diff edited this file for other cards without removing these sections.

### [LOW] Retired row names in help text and comments

**Location:** `internal/loomcli/validate.go:166`, `internal/battenshed/doc.go:33`, `internal/battenshed/notice.go:143`, `internal/loomshed/doc.go:52`.
**Issue:** The `--rework` flag survives by design (card 44's Intent), but its help text says PR-Rework's own gate runs it, and the other sites cite PR-Gate as a live row.
A reader following the text looks for a gate that no longer exists.

### [LOW] envkit keeps retired entries

**Location:** `internal/testkit/envkit/envkit.go:284`.
**Issue:** The fixture still declares a `Merge-Review` segment (card 23) and a `publish` gate list (card 48), so the fixture models a recipe and gate schema the code no longer has.

## Rejected

- Fork `code-tests`: no test pins the status-derived hand-back mark. `sheddriver/drive_test.go:149` wires `Deps.HandsBack` for a blocked Finalize reroute-cap status, which exercises that path.
- Fork `code-tests`: Finalize's deferred lock release lacks a test per exit. The single `defer` at `finalize.go:263` covers every exit, and tests assert release after the Stuck, recheck and landing paths (`finalize_test.go:930`, `finalize_recheck_test.go:75`, `:86`, `:156`).
- Fork `code-tests`: batten's `HandBack` closure at `battencli/wire.go:966` is untested. It is one line of wiring over the pinned predicate, and the predicate and its consumer each have a test.
- Fork `code-tests`: redundant `TestHandsBack` rows. The rows differ only in reason text, but they document the reasons the callers write and cost nothing; not worth a finding.
- Fork `code-correctness`: `NewMergeIn` leaves nil seams unchecked. These are internal dependencies filled by wiring, and `TestLandingDeps_EveryFieldPopulated` pins that fill, so code-quality's trust-the-caller rule applies.
- Fork `code-correctness`: `mergeIn` clears every source reason when `warpSHA` is set. `resolveMergeSources` raises only the warp-side source-not-found reason (`mergeguards.go:60`), which the explicit `CommitTree` check replaces.
- Fork `code-affected-set`: `verifytree.FailureKindPublish` stays declared. The plan does not call for its removal, and old records can still carry it.
- Declared PATTERN changes, commit trailers and the other focus items (lock-first Finalize, hand-back consumers, migrations, recipe tails) were checked by the forks and by me, and raised nothing.
