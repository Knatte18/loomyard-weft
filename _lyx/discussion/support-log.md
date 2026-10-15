## Interview

Autonomous mode: no operator answers; every pick below is an auto-pick recorded in the question ledger.
No advisors were seated.

### Turn 1 — exploration

Read the board entry (operator decision 2026-10-15), the board list, the master template, the fork and recovery prefixes, the FRESH-READ and card-read fragments, the verify-fix body, the orchestrator pattern directive, `internal/websterengine/doc.go`, `docs/overview.md`, the `aa925bdcc` commit message and the forkaudit header.
Found the codebase-read instruction in the master's opening paragraph and `## Orientation`, the inherit claim in the fork prefix, the recovery prefix and every FRESH-READ fragment, and the same claim in `doc.go` and `docs/overview.md`.
Wording pins sit in `claims_test.go` and `template_test.go`.

### Turn 2 — scope batch (auto-picked)

- Q1: What may the Master read? Recommended: `00-overview.md` and the `pattern/` backgrounds; no card files, no code. Alternative: also every card file, rejected (bloats every fork with other batches' cards).
- Q2: Touch the inline-layout fragment? Recommended: no, its banner pins it as the measurement baseline.
- Q3: Add a Go guard (hook or audit finding) on Master source reads? Recommended: no; the board entry asks for a stencil fix.
- Q4: Is the forkaudit re-measurement a card? Recommended: no; it needs runs on the deployed binary.
- Q5: New PATTERN entry? Recommended: no; one role's rule, and fans that read code before forking stay correct because nothing edits during a fan.

### Turn 3 — edge batch (auto-picked)

- Q6: A plan editing `PATTERN.md` or a `pattern/` file? Recommended: accept as an open risk, bounded by the card naming it as a target and the plan's Shared Decisions.
- Q7: The recovery strand renders the plain FRESH-READ fragment too; recommended: reworded text must hold for a session that inherited nothing.
- Q8: The verify-gate fixer fork? Recommended: say its inherited context holds no code.

### Lens review

Every fork returned a report.

- board-fit M, pins clash with `lighter-test-rules`: fixed, pins assert only what must hold (Constraints).
- board-fit L, `webster-deviations-list` and `plan-test-file-path` unnamed: fixed in Scope Out.
- premises L, plan drift premise imprecise: fixed in D1 rationale.
- premises L, `## Orientation` order pin is on the master heading: fixed in Notes.
- premises L and scenarios M, read set without PATTERN: fixed in D1 wording and a new acceptance criterion.
- premises L, D2 overstated the deadlock guard: fixed, the hook is named as the guard.
- goal-scope M, re-measure has no carrier: fixed with D8 (summary names it).
- goal-scope M, "doc files" ban contradicts D1: fixed in D1 and the acceptance criterion.
- goal-scope L, D4 rationale: fixed, cites the false claim.
- goal-scope L and scenarios L, "Read and Grep" line left broad: fixed in D1.
- goal-scope L, sibling entries unnamed: merged with board-fit L.
- pattern-fit M, `tools/sandbox/SANDBOX-WEBSTER-SUITE.md` missing: fixed in Scope and D6.
- pattern-fit L, master banner unnamed: fixed in Scope.
- pattern-fit L, test budget rule: fixed in Constraints.
- pattern-fit L, D7 consistent: no change needed.
- leanness L, re-measure stated three times and two Auto-mode bullets repeat Scope: fixed, bullets cut.
- leanness L, D7 fan sentence: moved to Rejected alternatives.
- leanness L, skills and color constraints padding: fixed, cut.
- leanness L, "forks explore more" risk: moved to the ledger.
- leanness L, render.go note: fixed, cut.
- scenarios M, inline layout contradicts the every-layout criterion: fixed, the criterion names the three live layouts and Scope Out says why the inline hedge stands.
- scenarios L, deploy mid-run mixes designs: fixed, Open risk added.
- scenarios L, a second fixer inherits a superseded findings file: rejected, the gate prompt already directs the fixer to the file the latest message names.
- scenarios L, no resume criterion: rejected, a resumed Master renders the same template file.

## Rejected alternatives

- Master reads every card file up front: every fork would carry every other batch's cards, and the per-card layout already reads the batch's own cards first.
- A PreToolUse hook or audit finding refusing Master reads outside the plan and rules: new Go behavior the board entry does not ask for; the stencil fix plus the post-landing audit measures compliance.
- Rewording the inline-layout fragment for uniformity: breaks its role as the layout measurement baseline.
- A PATTERN entry for "a forking session reads nothing its forks rewrite": one role is bound today; `stencil-intent-audit` is the place to find whether others need it.
  The Burler lens fan and twill forks read code before forking, and that stays correct there because no one edits during a fan.
- Extending `tools/forkaudit` with a stale-read-edit metric in this task: not asked for; the entry asks only that the re-measure compare that.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 Master read set — resolved, auto-pick: plan overview and rules only (D1).
- Q2 inline-layout fragment — resolved, auto-pick: untouched.
- Q3 Go guard — resolved, auto-pick: none.
- Q4 forkaudit re-measure — resolved, auto-pick: post-landing operator step.
- Q5 PATTERN entry — resolved, auto-pick: none (D7).
- Q6 plan editing the rules — resolved, auto-pick: open risk with bound.
- Q7 recovery fresh-read wording — resolved, auto-pick: wording holds for a cold session (D3, D4).
- Q8 verify-gate fixer — resolved, auto-pick: states no inherited code (D5).
- Q9 carrying the re-measure past landing — resolved, auto-pick after lens review: the summary names it (D8); the orch may also file a board note, which this writer cannot.
- Accepted cost, not a risk to act on: forks explore more by construction, so per-fork tokens and wall time rise.
