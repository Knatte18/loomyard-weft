# Support log — coverage map uses: reach note, failure triage and diff mutation check

## Interview

The session ran autonomously with no advisors, so every turn is a self-made pick, recorded in the question ledger.

### Turn 1 — the board and the study

Read the board entry `testmap-uses-design`, its project `testmap-uses`, the done research task `testmap-use-cases` and its report.
No `Decision for testmap-uses-design:` note exists.
`lym loom peers` lists one peer, `pause-after-crash-fixes`, which touches fabric and tender code and none of the gate or testcover code.

### Turn 2 — exploration

Two subagents mapped the maps' format, storage and readers, and the gate report, failure report, fixer brief and review flow.
Findings that shaped the design:

- `internal/testcover` owns the map, one JSON file per package; every reader reads one package at a time, and no cross-package index exists.
- The hub's shared maps sit under the board directory; a `--map` run off the prime writes the worktree's own maps under its anchor.
- The gate's `redundancy` report step is an advisory step after the tiers and lints, with a slot, budget, report file, red-gate and scoper notes, and a review note matched against HEAD's tree.
  Both new steps copy that shape.
- The failure report is printed by `internal/gatecli`; a cold fixer reads a report file its caller points it at, so lines inside the failure report reach the fixer with no stencil change.
- The finder of changed functions went with the selector, in `72b26d1a` and `b92c95ee`; only path-level change detection is left.
- No code records a "last green" commit for a parent branch.
- The mutant machinery plans, overlays, builds and runs per package, and treats a timeout or memory cap as unjudged.

### Turn 3 — design picks

Picks D1 to D12 as the ledger records.
The one pick that departs from the board entry's order is D11, the mutation trial: it is raised under Open risks for the parent.

### Turn 4 — lens review

All six forks returned a report. Findings, merged across lenses:

- Dirty tree: the `--map` write refuses a package with uncommitted files, so D3 got no fresh map on a writer's uncommitted `batch` gate (interactions F1, existing-paths 2, walkthrough 1, bound-breaker 1). Fixed: D3 refreshes only clean packages, and the note says which packages it answered from older maps and why.
- Refresh in the prime writes the shared maps (interactions F5, existing-paths 3, bound-breaker 8). Fixed: D3 always writes the worktree's maps and refreshes nothing in the prime.
- Redundancy step reusing the refreshed maps changes its input (interactions F2, bound-breaker 3, existing-paths 5). Fixed: the reuse sentence is dropped.
- D9 on a package without a fresh map reports false uncovered blocks (interactions F3, walkthrough 5). Fixed: D9 runs only on a package with a fresh map and reports the rest as not judged.
- `lym gate mutate` built no fresh map (interactions F4, walkthrough 4). Fixed: D11's verb runs D3's refresh first.
- Triage lines could break `gateslot.FailedTests` and the cap (existing-paths 1, bound-breaker 4, interactions F9, walkthrough 7). Fixed: D7 places the line as the test's first four-space line, counts it in the cap, and adds a parse criterion.
- The fixer brief resting on an unchecked report path (leanness 3). Fixed: checked that the card fixer reads the call's stderr (`fixAndRerun`); D7 now requires every other fixer's report to carry the lines. The Notes bullet is dropped.
- The D5 match ignored test files (walkthrough 2, bound-breaker 2). Fixed: the match also hashes the Go files, test files included, of the changed and named packages, with the remaining bound stated.
- D8 walked the run's branch, not the parent's (walkthrough 3, bound-breaker 5). Fixed: D8 walks the parent branch, read through fabric, with `--branch`.
- D8 command with no `--since` value (interactions F6, walkthrough 6). Fixed: placeholder `<last green commit>`.
- D8 output and map staleness (interactions F7). Fixed: it names the map commit, with the bound.
- D6 and D8 argument shapes (leanness 9). Fixed: both or neither are refused.
- Worktree map winning by location (bound-breaker 6). Fixed: it wins only when its file hashes match, otherwise the later-built map.
- Index stamp can miss a same-size rewrite (bound-breaker 7). Fixed: bound stated in D1.
- How an existing hub gets the template's reports (existing-paths 4). Fixed: D12 states that config load fills a template key a tuned entry lacks, as the walkthrough and bound-breaker lenses read in `internal/gateslot/doc.go`.
- Mutation order against redundancy (interactions F8). Fixed: D3 runs reach before any other report step.
- Trial order (board-scope 1, class `decision`). Kept as the Open risk for the parent, now with the darn cost; the writer cannot settle it.
- Unlisted step and `drop-unused-features` (board-scope 2). Fixed: added to that Open risk.
- Who supplies the last green (board-scope 3). Fixed: D8 says the caller, in practice the parent.
- Leanness 1, 2, 7, 8: Auto-mode assumptions, Constraints and Open risks cut to what the Decisions do not say; the Notes bullet that repeated `relevant.md` is dropped. Fixed.
- Leanness 4 (Goal repeats Scope): fixed, Goal cut to one paragraph and Scope trimmed.
- Leanness 5 (study figures in rationale): rejected, since each figure is the reason for its cap or filter, cut to one clause.
- Leanness 6 (survivor listing said twice): fixed, D10 refers to D9.

## Rejected alternatives

- **An index written by every reader into the shared map directory.** Rejected: a worktree's gate would then write the hub's shared directory, which today only the prime's map run writes.
- **No persistent index, an in-memory table on every call.** Kept only as the fallback: the board entry asks for an index, and the verb and the gate would each read up to 56 MB.
- **A changed-function finder by line ranges of the diff.** Rejected for D2: the maps name functions outside the own package, so function text is the matching grain. Line ranges are kept for D9, where blocks have lines.
- **Triage lines added to each fixer stencil.** Rejected: the fixer already reads the failure report, so the lines reach it there.
- **A "last green" record for the parent branch.** Rejected: no code keeps one, and adding it is a project of its own; the caller names `--since`.
- **Running the mutation trial inside this run.** Rejected: it needs a worktree per landed commit, which no implementer of the run may create.
- **Cutting the mutation check from this run until the trial reports.** Not chosen by this writer, since the task asks to design all three uses; it is offered to the parent under Open risks.
- **Mutating string constants and embedded lines on changed lines.** Rejected: they produce the mutants reviewers call unproductive, such as message text.
- **Making the tests-per-function cap and the commit-range cap config keys.** Rejected under YAGNI; they are named constants.
- **The redundancy step reading the maps the reach step refreshed.** Rejected after the lens review: a fresh worktree map holds the diff's own new tests, which changes what the redundancy step judges against.
- **A dirty-tree map measured in memory and never written.** Rejected: it would change how `--map` measures a package, which is out of scope; D3 refreshes clean packages only and says so for the rest.
- **D8 walking the current branch.** Rejected after the lens review: in a run worktree the parent's landings sit behind a merge's second parent.

## Review rounds

### Round 1

- F1: D3 adds the reach turn, the channel by which the loomshed verify gate and the weaver's batch gate hand a passing verify's note to their writer, once per HEAD tree. Dropping the writer from the note's purpose was the other path; it would leave the operator's ask unmet.
- F2, F5, F14: D7 drops triage lines before any test output is cut, says no change set exists when D2 has none, and puts the D8 command on its own line.
- F3, F13: D3 also checks linked packages for uncommitted files; the batch Open risk covers only a hand-run gate.
- F4: D4 lists a new function that older cross-package maps may reach as new since those maps.
- F7, F8, F9, F10, F11, F12: D9 and D11 judge from matching hub maps in the prime; D1 stamps the set of map files; D2 matches by package and name over two given sets of contents; D5 never matches a red-tier report; D6 refuses `--since` or `--branch` without `--test`.
- F15: the repeated auto-mode assumption is cut.
- F6 (trial order, class `decision`) is with the parent.
- Self-check: the reach turn now fires once per unreached function rather than once per tree, and each gate is told its attempt budget. D4 keeps a function that older cross-package maps may reach in the unreached list, with a caveat, so the turn and the review still see it. D2 matches a name repeated across build-constrained files by file. D1 states the rebuild bound after an interrupted map run.

## Question ledger

- Q1 (auto-pick): Where does the cross-package reader live? Picked a new package `internal/testreach`, beside `internal/testcover`, which keeps the map format. Rationale: the reader, index and finder serve three callers and are not part of the map's format.
- Q2 (auto-pick): Who writes the index? Picked the prime's `--map` run only, readers falling back to an in-memory table. Rationale: keeps the shared directory written only from the prime.
- Q3 (auto-pick): Which map wins for a package both directories hold? The worktree's when its file hashes match the worktree, otherwise the later-built one. Rationale: a worktree map goes stale once HEAD moves.
- Q4 (auto-pick): How is "most specific" ranked? By the number of functions a test reaches. Rationale: the only breadth measure the maps hold.
- Q5 (auto-pick): Which gate keys list `reach`? Every key listing `redundancy`, plus `batch`. Rationale: the operator asked that the writer get the note from its own gate; `batch` is a tapestry writer's gate.
- Q6 (auto-pick): Does the reach step build fresh maps? Yes, of the clean changed packages, into the worktree only, under `reach_budget_min`, skipping packages whose map already matches, and none in the prime. Rationale: the board entry's "fresh map of the diff's packages"; the `--map` write refuses a dirty package and writes the shared maps in the prime.
- Q12 (auto-pick): Which branch does D8 walk? The run's parent branch read through fabric, or `--branch`. Rationale: from a run worktree the parent's landings are not on the current branch's first-parent line.
- Q7 (auto-pick): How does triage reach the fixer? Through the failure report the fixer already reads. Rationale: no new channel.
- Q8 (auto-pick): Where does "last green" come from for a red parent? The caller passes `--since`; the verb refuses without it. Rationale: no such record exists.
- Q9 (auto-pick): What filter and cap does the mutation check start with? One mutant per changed line, an operator order, no string, embed or logger mutants, a per-diff cap and a budget. Rationale: Google's finding that a cap and a filter belong in the first build.
- Q10 (auto-pick): How is the trial handled? The step ships unlisted, a verb runs it by hand, and a board note asks for the trial and the listing decision. Rationale: the trial needs worktrees per landed commit. Raised under Open risks for the parent.
- Q11 (auto-pick): Which PATTERN entries change? A new `PATTERN-coverage-map-reader`, and `PATTERN-test-speed`, whose background says no gate builds a map. Rationale: D1 and D3.
