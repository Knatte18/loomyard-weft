---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/02-bouncer-carry-over.md:6
    summary: Edit cards 1, 2, 3, 5, 7, 8 and 9 list bare non-test Go file paths while their Intent names specific symbols, so no member glyph is listed
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/plan/09-resume-circling.md:26
    summary: PendingCirclingDecision folds a settled decision into the no-decision case, so resume over an awaiting Bouncer row with a settled accept refuses toward circling verbs that refuse back toward resume
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/09-resume-circling.md:29
    summary: The decision-unreadable refusal has one way forward, fix or delete the decision file, but PendingCirclingDecision can also fail on the run directory or a report file, which that way forward does not fit
  - id: F4
    severity: LOW
    class: scope
    location: _lyx/plan/09-resume-circling.md:3
    summary: D6 says every way forward over an awaiting Bouncer row names resume and the circling verbs, but battenshed's awaiting hand-off text names approve or reject and no card edits it or records the carve-out
---

### [MEDIUM] Edit cards name symbols in their Intent but list only Go file paths

**Location:** `_lyx/plan/01-carry-over-write.md`, `02-bouncer-carry-over.md`, `03-carry-over-wiring.md`, `05-lighter-convergence-rule.md`, `07-segment-models.md`, `08-fix-start.md`, `09-resume-circling.md`, each card's `**Edit:**` list.

**Issue:**
Each of these cards edits existing non-test Go files and lists them only as file paths, while its `**Intent:**` names the symbols that change.
Card 2 lists `internal/shedadapters/bouncer.go` and names `BouncerConfig`, `NewBouncer`, `settle` and `settleUnconverged`.
Card 3 lists `internal/shedrecipe/recipe.go`, `entries_bouncer.go` and `internal/loomcli/wiring.go` and names `Env`, `bouncerEntry` and `wire`.
Card 5 lists `bouncerfacts.go` and `bouncerprompt.go` and names `roundFacts`, `renderRoundFacts` and `decisionRuleMarker`.
Card 7 lists `config.go`, `review.go` and `entries_burler.go` and names `Config`, `LoadConfig`, `ResolveReview`, `ReviewSettings` and `burlerRoundEntry`.
Card 8 lists `engine.go`, `handoff.go` and `profile.go` and names `Engine.Run`, `join` and `RunOpts`.
Card 9 lists `resume.go`, `circling.go` (both packages) and `bouncer.go` and names `resumeFromRunState`, `circlingWayDecided` and `escalationReason`.
Card 1 lists `append.go` and names `AppendDecision`.
This is one method gap across the plan, so it is filed once; it is not a per-card list of misses.
A file-only target gives webster's changes-files guard and the card's reviewer no symbol-level scope.

**Fix:**
List the member glyphs in each affected `Edit` group, in addition to the test, YAML and markdown paths that stay file paths.
For example, card 2 takes `internal/shedadapters#BouncerConfig`, `internal/shedadapters#NewBouncer`, `internal/shedadapters#Bouncer.settle` and `internal/shedadapters#Bouncer.settleUnconverged`, and card 9 takes `internal/loomcli#loomCLI.resumeFromRunState` and `internal/shedadapters#escalationReason`.
Skip the file self glyph for a file whose member glyph is listed (`redundant-file-target`), and keep a file path only where the edit is a comment or header with no symbol behind it.
Add the re-sign arrow to a member only if its parameters, results or receiver change; none of the Intents above says so except possibly `join` in card 8, whose restructure should state whether its signature changes.

### [MEDIUM] A settled circling decision is treated as no decision

**Location:** `_lyx/plan/09-resume-circling.md`, the `PendingCirclingDecision` bullet and the `resume.go` bullet on awaiting at a Bouncer row with no pending decision.

**Issue:**
`PendingCirclingDecision` returns `false` both when the latest round has no decision file and when its accept is settled (the card's own test row: "false for a settled accept").
The `resume.go` bullet maps `false` to the refusal "awaiting at a Bouncer row with no pending decision refuses, naming `lyx loom circling accept <slug>` / `continue <slug>` and then `lyx loom resume`".
A settled accept in a run still `awaiting` is a recorded decision.
It is reachable if the driver stops after `settleCirclingAccept` writes the file and before the status moves off `awaiting`.
In that state the circling verbs refuse with `circlingWayDecided`, which this card moves to `lyx loom resume`, and resume refuses toward the circling verbs: the two refusals send the operator to each other.
`lyx loom start` is the step that works, since the Bouncer's Stuck path for an already settled accept names it, but neither refusal says so.
This is the focus question on whether each refusal has a way forward that fits its cause: the no-decision refusal is right for no decision and wrong for a settled one.
D6 scopes the wake to a "recorded, not yet settled" decision and names a refusal only for "no recorded decision", so the settled case is left unspecified.

**Fix:**
Have `PendingCirclingDecision` distinguish the three states (none, pending, settled), or add a settled result, and give the settled case its own refusal and spec row whose way forward is `lyx loom start` in the task worktree, which re-enters the segment through the Bouncer's already-settled path.
Add a `resume_test.go` row that reaches it, and extend the `circling_test.go` (shedadapters) rows to pin the third state.

### [LOW] The decision-unreadable way forward does not fit every error PendingCirclingDecision returns

**Location:** `_lyx/plan/09-resume-circling.md`, the `resume.go` bullet on a malformed or unreadable decision file, and the `refusal-spec.md` bullet's new `resume: circling decision unreadable` row.

**Issue:**
The card gives the refusal one way forward: "fix or delete the named decision file, then re-run `lyx loom resume`".
`PendingCirclingDecision(runDir)` reads the latest round the way `RecordCirclingDecision` does, through `ResolveRound`, which can fail on the run directory (`stat run dir`, not a directory) or on a round report file (`stat report file`) before any decision file is reached.
Those errors name a directory or a report file, not a decision file, so the way forward points at a file the message does not name.
The card says which errors the refusal covers only as "a malformed file is an error" and "a malformed or unreadable decision file".
After a delete, the re-run reaches the no-decision refusal, which then names the circling verbs; that works but the first refusal does not say it.

**Fix:**
State in the card which errors the row covers.
Either split the run-directory and report-file errors into a transient row whose way forward is to fix the path the message names and re-run `lyx loom resume`, as `circling: decision record failed` does, or reword the row's way forward to "fix the file or directory the message names; for a malformed decision file, delete it and record the decision again with `lyx loom circling accept <slug>` / `continue <slug>`".
Add the row to `resume_test.go` if split.

### [LOW] D6's every-way-forward clause leaves battenshed's awaiting text unaddressed

**Location:** `_lyx/plan/09-resume-circling.md`, the **Bound** paragraph ("batten's decision reading are unchanged"), against `internal/battenshed/innerrun.go` `awaitingHandOff`.

**Issue:**
D6 requires "every way-forward over an awaiting Bouncer row name `lyx loom resume` and the circling verbs, never approve or reject".
`battenshed`'s wait over an awaiting child reports `awaitingHandOff` ("run `lyx loom approve` or `lyx loom reject` in the task worktree; this run then resumes the child itself") whenever no PR-Gate decision exists, and it does so for a child escalated at a Bouncer row as well.
`approve` refuses at a Bouncer row, so that text is a dead end in the very case D6 describes.
The card leaves batten's decision reading untouched, which D6 and the Out list license, but it neither edits this text nor states that the text is an accepted exception to the every-way-forward clause.
The card's `docs/overview.md` edit tells the reader a batten-driven run is resumed with `lyx loom resume`, so the docs and batten's own report disagree.

**Fix:**
Decide and record it in the card.
Either add `internal/battenshed/innerrun.go` (the `awaitingHandOff` constant and its use) to the Edit list so a child awaiting at a Bouncer row reports the circling verbs and `lyx loom resume`, leaving `ReadDecision` unchanged, or state in the card's Intent that batten's hand-off text is a named exception and why, so the every-way-forward clause is not read as covering it.

## Focus verdicts

These settle the five questions the directive posed; none is a finding beyond those above.

1. **Card 9 refusals.**
   The no-decision, decision-unreadable and recipe-lookup refusals each have a `refusal-spec.md` row in the card and a `resume_test.go` row that reaches it.
   The recipe-lookup way forward (rebuild or reinstall lyx, then re-run) matches the circling verbs' own `circlingWayRecipe`.
   The malformed-file way forward is delete-or-fix, not a bare re-run, as the directive asks.
   What does not fit is covered by F2 (settled decision) and F3 (errors that are not about a decision file).
2. **`Env.CarryOver` and `Env.RowReviewModels`.**
   Both are covered.
   Card 3 lists `CarryOver` in `battencli`'s `intentionallyNil` in `wire_seams_test.go`, which is the file that keeps that package's `TestWire_EverySeamFilled` green, and loomcli fills it.
   Card 7 adds `RowReviewModels` to `envkit`'s `nilLegal`, so `envkit.NilSeams` skips it for both packages while loomcli still fills it.
3. **Resume wording sites.**
   `internal/loomcli/cli.go` (the `circling` paragraph of the root `Long`) and the `docs/overview.md` review-segment escalation paragraph, including its batten sentence, are both in card 9's edit list.
   The only other site over an awaiting Bouncer row that I found is batten's hand-off text, raised as F4.
4. **Card 1 result check.**
   The card's check covers all three parts of D2's bound: the H2 sequence, at most one well-formed pair per segment and exactly one for the written one, and every line outside the replaced span.
   It defines the replaced span separately for replace, append (the appended blank line and entry) and removal (the removed blank line and entry), so the append and removal cases are covered.
5. **Card 6 and `escalation_test.go`.**
   The card does not edit that file, and its claim is true of what the test reads: `escalation_test.go` sets its own budget constant (2), never reads the template, and asserts that the budget escalation carries cause `budget`.
   The template value of 3 is pinned in `config_test.go`, and the budget reaches the routing through `loomcli`'s `wiring_test.go`.
   The acceptance line "a CONTINUE at round 3 escalates with cause `budget`" therefore holds by composition of those tests and no single test drives the template value through a Bouncer; I treat that as sufficient and raise no finding.
