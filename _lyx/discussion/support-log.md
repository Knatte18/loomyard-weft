# Support log — creel-sandbox-suite

## Interview

### Turn 1 — exploration and batch 1 (autonomous, no advisors)

Read the board entry and the related creel notes (`creel-notices`, `creel-agents`), the shuttle suite, `tools/sandbox/suite.go`, `tools/sandbox/main.go`, their tests, the launchers, `docs/sandbox-howto.md`, the coverage guard and the creel daemon lifecycle in `internal/creelengine/doc.go`.

Batch 1, each settled by auto-pick:

- Q1 Subcommand and launcher name. Recommended: `creel-suite`. Picked.
- Q2 Renumber S9 into the C series? Recommended: keep ids, as the entry says. Picked.
- Q3 `reedTeardown` for the creel suite? Recommended: true, since S9 and the C fixtures add strands. Picked.
- Q4 Change the coverage guard? Recommended: no, it globs every suite file. Picked.
- Q5 The shuttle session log omits S4 to S7: fix while rewriting it? Recommended: yes. Picked.
- Q6 Stop the creel daemon after a session? Recommended: no, out of scope and unchanged from today. Picked.

### Turn 2 — lens review

Every fork returned a report.

- premises LOW, D2's claim that past reports name S9 and the C ids rests on nothing read: fixed, clause cut (merged with leanness 7).
- premises LOW, the wired-worktree need was attributed to creel too: fixed, D1 names it as reed's.
- board-fit MEDIUM, `reed-add-parentless-placement` sweeps the parentless `reed add` lines being moved: fixed, Open risk added.
- board-fit LOW, the #673 `--anchor own-window` is not added to moved lines: fixed, Scope Out says so.
- goal-scope LOW, D5 departs from the entry's "guard follows the move" wording: rejected, D5 already explains the guard follows by its glob.
- goal-scope LOW, the S4 to S7 log fix is extra scope: rejected, it sits in a log D3 rewrites and is stated.
- goal-scope LOW and pattern-fit LOW, the howto's suite link and `PATTERN-markdown-link-integrity`: fixed, Scope, Constraints and criterion name the resolving link.
- goal-scope LOW, `main.go`'s comment lists subcommands by name: fixed, plan-writer note covers name lists.
- pattern-fit LOW, prose criteria may invite a text-pinning test: fixed, Constraints say suite text is checked by review.
- pattern-fit LOW, no line says PATTERN is unchanged: rejected, no decision touches PATTERN and such a line is padding.
- leanness LOW, Constraints restated general rules: fixed, trimmed to coverage, link integrity and test budget.
- leanness LOW, the coverage-guard point was made three times: fixed, kept in D5, Constraints points to it, Scope Out line cut.
- leanness LOW, daemon stop in both Scope Out and Open risks: fixed, kept in Open risks.
- leanness LOW, Auto-mode assumptions repeated D3 and held a scope boundary: fixed, the D3 repeat cut and the sibling-Notes boundary moved to Scope Out.
- leanness LOW, the gate-pass criterion is a standing rule: fixed, cut.
- leanness LOW, a plan-writer note duplicated `relevant.md`'s test names: fixed, cut.
- leanness LOW, D1's section list could be derived: rejected, the entry asks for exactly those sections and D1 carries the `ref` list and S2 exclusion.
- scenarios MEDIUM, D4's new subcommand states no bound: fixed, D4 states what `runSuite` does and nothing more.
- scenarios MEDIUM, the absent-hub failure path is missing from the criteria: fixed, added to the table-test criterion.
- scenarios LOW, nothing pins `shuttleSuite` keeping teardown: rejected, `TestSuiteSpecs_ReedTeardownFlag` already pins it and no Decision changes `shuttleSuite`.
- scenarios LOW, the daemon risk on Windows `build -reset`: fixed, Open risk amended.
- scenarios LOW, back-to-back suite runs lose an unfetched report: fixed, Open risk added.

## Rejected alternatives

- Renumbering S9 as C0 or C14: breaks references to S9 in past reports and issues, and the entry asks for ids kept.
- A creel suite that points back to the shuttle suite for its preamble: the agent reads only the one copied file, so each suite is self-contained.
- `reedTeardown: false`: leaves a tmux server that blocks the next `build -reset`.
- Adding a creel-daemon stop to the launcher: no sanctioned stop verb exists besides a binary replacement, and no scenario asks for it.
- Carrying the shuttle suite's controlled tmux exception into the creel suite: no creel scenario needs it, and its socket lookup depends on a shuttle run's strand guid.
- Updating every sibling suite's Notes list: none lists every suite today, so adding creel to each is churn with no reader need.

## Review rounds

_No rounds yet._

## Question ledger

- Q1 Subcommand name — resolved, auto-pick: `creel-suite`, by the `<module>-suite` naming every launcher uses.
- Q2 Scenario ids — resolved, auto-pick: kept, per the board entry.
- Q3 Reed teardown — resolved, auto-pick: true, the suite adds strands.
- Q4 Coverage guard — resolved, auto-pick: unchanged, it globs `tools/sandbox/*SUITE.md`.
- Q5 Shuttle session log omission — resolved, auto-pick: list S1 to S8.
- Q6 Creel daemon after a session — resolved, auto-pick: out of scope, recorded as an open risk.
