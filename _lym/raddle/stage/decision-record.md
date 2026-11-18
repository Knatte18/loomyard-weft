# Decision record — coverage map uses: reach note, failure triage and diff mutation check

## Goal

Put the per-test coverage maps of `internal/testcover` to the three uses the map study ranked worth building (`docs/research/speed/testmap-use-cases-2026-11-18/report.md`), over one reader of the maps:
a reach note for a diff, for the writer and the code review; failure triage in the gate's failure report and the fixer's brief; and a mutation check of the lines a diff changes.
Build order: the reader and the reach note, then failure triage, then the mutation planner and `lym gate mutate` (D9, D11), then the trial,
then the `mutation` step (D10) only if the trial pays.
This run builds up to `lym gate mutate`; the trial and D10 come after it.

## Scope

In:

- A new package `internal/testreach` (new): the one cross-package reader of the maps, its index, and the finder of a diff's changed functions.
- The `reach` report step of `lym gate test`, its report file and its note in the code review rubric.
- The `mutation` report step and its rubric note (D10), designed here and built after the trial.
- The reach turn of the loomshed verify gate and the weaver's batch gate (D3).
- The verbs `lym gate reach` and `lym gate mutate`.
- Triage lines in the failure report.
- A changed-line mutant planner in `internal/testcover`.
- A board note for the mutation trial (D11).
- The `gate.yaml` template's `reports` defaults and the new config keys.

Out:

- Any test selection or ordering from the maps; it stays retired.
- Maps of tier 2 or tier 3 tests, or of spawned binaries.
- The study's other uses: the pseudo-tested sweep, the drift signal, suspiciousness ranking and the rest.
- Any change to how `lym gate test --map` measures a package.
- Any language other than Go: like the redundancy step, the new steps write a note naming the scoper for any scoper but `go`.

## Decisions

### D1 — One reader of the maps, with an index

`internal/testreach` (new) is the only production code that reads coverage maps across packages.
It answers two lookups: the tests that reach a function, and the functions of a given set that a test reaches.
A function is a `testcover` function key: import path, file and function name, a method named `<Type>.<Method>`.

- It reads two map directories through `testcover.Dir`: the hub's shared maps under the board directory and the worktree's own maps under its anchor.
  For a package both hold, the worktree's map wins when its recorded file hashes match the worktree's files; otherwise the map built later wins.
- In its own package a test reaches a function when it covers one of the function's blocks;
  in another package, when the map lists the function among the test's linked functions.
  An entry with an outcome or a spawn mark reaches nothing.
- The index is a reverse table from function to reaching tests over the hub's shared maps, a cache file beside them.
  It is stamped with the set of map files in the directory and each one's size and modification time,
  so an added or removed map file is a stamp difference.
  `lym gate test --map` run from the prime writes it after its maps; no other caller writes it.
  A reader whose index is absent or whose stamps differ builds the same table in memory from the maps and goes on.
  The worktree's own maps are few and are read whole, never indexed.
- Every answer carries the commit of each map it rests on.

Rationale: the lookups read up to 56 MB of JSON across packages, and all three uses ask the same two questions.
Writing the index only with the prime's map run keeps the shared directory written from the prime alone, as today.
Bound: a map rewritten at the same size within one modification-time tick keeps its stamp, so the index can serve its old pairs until the next map run; every answer is advisory.
Bound: after a map run that ended before its index write, as a moved HEAD ends it, every reader builds the table in memory until a map run writes the index again.

### D2 — Changed functions by function text

`internal/testreach` (new) finds the functions a diff changes by comparing two given sets of file contents, the base and the head, function by function.

- A function is matched across the two sets by package and name, whatever its file, so a function moved verbatim to another file of its package is unchanged.
  A name the package defines in more than one file, as build-constrained variants do, is matched by file instead,
  and the `init` functions of one file are matched in their order in that file.
- A function is changed when it is new or its text differs; a deleted function is not a changed function.
- Only production Go files count.
- The caller hands in both sets; `internal/testreach` (new) runs no git.
  The reach step, the failure triage (D7) and the mutation check (D9) hand in the files at the merge base with the parent branch, as the redundancy step takes it,
  read through `internal/fabricengine`, as the base, and the worktree's files as the head.

Rationale: function text is the grain the maps hold outside the own package, and the old finder went with the selector.

### D3 — The `reach` report step

A gate key whose `reports` lists `reach` runs a step after its tiers pass, before any other report step.

- It finds the diff's changed functions (D2) and refreshes the worktree's own maps of the packages that hold them, with the `--map` measurement, inside a gate slot.
  The refresh always writes the worktree's anchor maps, never the hub's shared ones; a call in the prime refreshes nothing and answers from the shared maps.
- A package is refreshed only when its worktree map's file hashes differ from the worktree's files,
  and only when neither it nor any in-module package its test binary links has a modified or untracked file.
  A map is written only when no file the package or a linked package owns is modified or untracked (`PATTERN-test-speed`), so a refresh past either check would write nothing.
  The linked packages are those the package's map records, or, for a package no map holds, the in-module packages in the Go toolchain's dependency list of its test binary.
- `reach_budget_min` bounds the refreshes.
- A package left unrefreshed, because it or a linked package has uncommitted files, it lies past the budget or the call runs in the prime, is answered from the maps there are.
  The note says so for each such package, naming the dirty package when it is a linked one:
  for uncommitted files, the way forward is to commit and run the gate again.
- It writes `reach.json` in the worktree's gate directory through `internal/state`, recording what D5 matches on.
- It prints the note of D4 to stderr, which a writer reads from a gate call it runs itself.
- It is advisory: nothing it meets changes the exit of the `lym gate test` call.
  A red tier, a call outside a hub and a scoper other than `go` give a report that judges nothing, with a note saying which, as the redundancy step does.
- The reach turn carries the note to a writer whose gate runs after its turn through `verifytree.Verify`:
  the loomshed verify gate, which Darn-Write and Code-Burler-Fix run, and the weaver's batch gate.
  After a passing verify, such a gate reads `reach.json`.
  When the report matches HEAD (D5) and lists an unreached function (D4) that no earlier reach turn of this gate listed, the gate returns one failing result whose findings are the note.
  The findings tell the writer to add the missing tests and commit them, or to end its turn unchanged.
  The functions each turn listed, with their text hashes, are recorded in the gate's verify directory, so a function listed once never brings another turn.
  The next arrival runs the verify as usual and, with no new unreached function, passes or fails as that verify does.
  The turn spends one of the gate entry's attempts.
  Each gate is told its entry's attempt budget and counts its failing shuttle arrivals since its last pass;
  when one more failing arrival would reach the budget, it passes without the turn.
- A reach turn's failing result goes to the writer alone.
  Darn-Write's verify entry carries a cold fixer, `darn.gate_fixer`,
  which `gatefixer.WrapRowGate` hands a failing result before the writer hears of it.
  The wrap gains a check on the entry's own gate closure, and on its forked closure when one is set:
  it spawns no cold fixer for a result whose findings are the reach note.
  Today's `SkipForked` closure does the same for a forked failure alone,
  and Darn-Write sets no forked closure, so it cannot carry this bypass.
  The bypass holds only for such a result, so every other verify failure still goes to the fixer first.
- On an arrival that spawns a fixer, the wrap runs the inner gate a second time over the fixer's tree.
  The wrap marks that second run as a re-run, and the gate adds no failure to its count for a re-run,
  so the count of the attempt-budget rule above is one per shuttle arrival.

Rationale: the operator asked that the writer get the note from its own gate before review, so it adds the missing tests itself.
A writer that runs its gate by hand reads the note on stderr.
The Darn-Write, Code-Burler-Fix and weaver batch gates run after the writer's turn through `verifytree.Verify`, which keeps a passing run's stderr in its log and returns no findings,
so the reach turn is their channel.
`verifytree.Verify` refuses a dirty tree, so those gates' packages are always refreshable.
Bound: the step costs the map build of each changed package, each tier 0 and tier 1 test run alone; the budget caps it, and an unchanged package costs nothing on the next call.
Bound: the reach turn costs one writer turn and one gate attempt per arrival whose report lists an unreached function no earlier turn listed.

### D4 — What the note says

For each changed function the note names the tests that reach it:

- tests of the function's own package first, then tests of other packages;
- within each group, the most specific first: the tests that reach the fewest functions in their maps;
- at most five tests per function, then the count of the rest;
- the commit and age of each map it read.

It then lists the changed functions no mapped test reaches.
For each one whose package holds a tier 2 test or a spawn-marked map entry, it says that such a test may run it unseen.
For each one new since the base whose package another package's map records as linked with file hashes that differ from the worktree's,
it says that the tests of those packages, which it names, may reach it, since their maps predate it and D3 refreshes only the packages holding changed functions.
The note is capped in lines; the rest stays in `reach.json`, which the note names.

Rationale: the study counts 153 functions reached by 500 tests or more, so an uncapped list is noise.

### D5 — The reach note in the code review

The code review rubric gets a `reach_report` marker beside `redundancy_report`, filled as that one is: read at each fill, used only when it matches HEAD's tree, and otherwise a line saying no current report exists and why.
The report matches HEAD when HEAD's changed functions and their text hashes are those it records.
The Go files, test files included, of each changed package and of each package whose tests the note names must also hash as it records.
A report written on a red tier, outside a hub or under another scoper never matches, and the line then names that reason.
The rubric reads the note as evidence for its finding "a behavior the diff adds with no coverage", and for naming a covering test in a redundancy finding.
An unreached function is a candidate, never a finding by itself.

Rationale: a name search misses tests that reach a function through its callers.
Matching on test files too means a report written before the writer added a test no longer matches.
Bound: a new test in a package the note does not name leaves the report matching.

### D6 — The read verb `lym gate reach`

`lym gate reach <file>#<Func>` prints the note of D4 for one function, from the stored maps and the worktree's maps, with no build and no gate slot.
The worktree-relative file and the function name, `<Type>.<Method>` for a method, name the function.
A function the maps do not hold gives a line saying so and the map's commit, with exit 0.
It writes nothing, not even the index.
A call that gives both a function and `--test` (D8), or neither, or gives `--since` or `--branch` without `--test`, is refused, naming the two forms.

Rationale: a writer asks before it edits which tests its change will hit, across packages.
Bound: it only reads, from maps that may be days old, and it says how old.

### D7 — Failure triage lines

When a `lym gate test` call fails, each failed test that a map holds gets one line in the failure report.
The line names the run's changed functions (D2) the test reaches, at most five and then a count, or says "reaches none of this run's changes".
A test no map holds gets "not in the coverage maps", and a failed subtest is looked up by its top-level test.
The line names the commit of the map it read.

- The line is the first four-space line under its test's two-space line, so the report's parsers read it as that test's output.
  `gateslot.FailedTests` and the reruns it drives parse the report unchanged.
- A line that says "reaches none of this run's changes" is followed by a second four-space line holding the D8 command, so the line's byte cap never cuts the command.
- A triage line never causes a cut.
  It is added, with its command line, only while the report's line budget still holds them, and triage lines are dropped before any test output line is cut.
- When the call has no parent branch, or its merge base cannot be read, D2 has no change set.
  Each mapped failing test's line then says no change set exists and why, never "reaches none of this run's changes", and carries no D8 command.
- A lookup that fails adds no line.
- Each fixer's report carries the lines.
  The card fixer's report is the call's stderr, which holds the failure report (`fixAndRerun` in `internal/gatecli`).
  A fixer whose report file does not copy the failure report gets the lines of its failed tests added to that file.

Rationale: all four flaky failures the study could check reached none of their landing's files; the line tells a fixer that before it chases one.
Bound: the lines are advisory; a stored map may lack a function the run added, and a stale entry can show a reach the test no longer has.

### D8 — Triage over landings for a red parent branch

`lym gate reach --test <package>.<Test> --since <rev>` walks the first-parent commits of the parent branch after `<rev>`.
For each commit it lists the changed functions (D2, with the first parent's files as the base and the commit's as the head, both read through fabric) that the test reaches.

- The parent branch is the run's parent branch, read through fabric as the redundancy step reads it, or `--branch <name>` when given.
  A worktree with no parent branch walks its current branch.
- Commits whose changes the test reaches none of are left out; the output says how many were checked and names the map commit.
- `--since` is required; the refusal names the flag and says to pass the last commit the caller knows was green.
- The walk is capped at 200 commits; a longer range is refused with the cap and the way forward, a later `--since`.
- A D7 line that says "reaches none of this run's changes" is followed by this command for the test on its own line, with `<last green commit>` as the `--since` value for the caller to fill.

Rationale: a repeated failure that is not the run's own costs a round trip to the parent to test the base; this names the landings to suspect without a build.
lym records no last green commit for a parent branch, so the caller, in practice the parent, names it.
Bound: an old landing's changed function that has since been renamed or removed matches nothing in the current maps.

### D9 — Mutants on the changed lines

`internal/testcover` gains a planner that places mutants only on the lines the diff adds or changes, in blocks a test covers.

- It runs only for a package with a fresh map: one D3 refreshed, or a worktree map whose file hashes match the worktree.
  In the prime, where D3 refreshes nothing and no anchor maps exist, a hub map whose file hashes match the worktree is fresh.
  Any other changed package is reported as not judged, with the reason, and gets no mutant.
- A block's lines come from the coverage binary's block table; the changed lines come from the same base and worktree files as D2.
- A changed block no mapped test covers gets no mutant and is reported as uncovered.
- Operators: a negated condition or comparison, a deleted operand of `&&` or `||`, a dropped statement and a shifted integer constant.
  String constants, embedded file lines and a dropped call into `internal/logger` get no mutant.
- At most one mutant per changed line, chosen in the operator order above.
- `mutation_max_mutants` caps the mutants of one diff, spread over the changed functions in turn.
- Each mutant runs the tests of the fresh map that cover its block, alone, until one fails.
  A run that ends at its time bound or the memory cap is unjudged, as `lym gate redundant` treats it.
- A mutant every covering test passes is a survivor, reported with its file, line and operator.
  A survivor whose function D1 shows reached by tests of other packages lists those tests as possible killers; they are not run.

The existing overlay, mutant binary and runner are reused, one binary per changed package.

Rationale: the study's sources put a cap and a filter in the first build, since developers called most unfiltered mutants unproductive.
The map cuts each mutant's runs from the package's whole set to the tests covering its block.

### D10 — The `mutation` report step and its review note

D10 is built only after the trial pays, in a later darn the parent opens from the decision the D11 board note asks for.

A gate key whose `reports` lists `mutation` runs the check of D9 after the `reach` step, inside a gate slot, bounded by `mutation_budget_min`.

- It writes `mutation.json` in the worktree's gate directory, matched against HEAD as D5 matches.
- It prints one summary line to stderr: mutants run, killed, survived and unjudged, blocks uncovered and packages not judged.
- The code review rubric gets a `mutation_report` marker, filled as D5's is, listing D9's survivors, uncovered blocks and packages not judged.
  A survivor is evidence for "a behavior the diff adds with no coverage"; the reviewer judges whether its change matters.
- It is advisory and judges nothing on a red tier, outside a hub, or with another scoper, as D3.
- The config check refuses a key that lists `mutation` without `reach`, naming `reach` as the fix.

Rationale: the check needs the fresh maps D3 builds; the refusal keeps that order in config.

### D11 — The mutation trial

`lym gate mutate` runs D3's refresh and then D9 over the worktree's diff, inside a gate slot under `reach_budget_min` and `mutation_budget_min`.
It prints the survivors, uncovered blocks and packages not judged that D9 finds, in the form D10 names.
`--base <rev>` picks another base.
In the prime the refresh does nothing, and D9 judges the packages whose hub maps match the worktree.
The run files a board note asking for the trial the board entry describes: about twenty landed diffs replayed with `lym gate mutate`, counting survivors and reading a sample.
The note also asks the parent, from the trial's counts, whether to build D10 and list `mutation` in the template's diff keys.

Rationale: the trial needs the planner and runner D9 builds, and a worktree per landed commit, which no implementer of this run may create.
The `mutation` step is not built before the trial says it pays.

### D12 — Template defaults and config

- The `gate.yaml` template lists `reach` first among the reports of every key that lists `redundancy`, and in `batch`, the gate of a tapestry's writers.
- New keys beside `redundancy_max_tests` and `redundancy_budget_min`: `reach_budget_min`, `mutation_max_mutants` and `mutation_budget_min`, with defaults in the template.
- The config check accepts `reach` as a report name, alongside `redundancy`.
  Its acceptance of `mutation`, and D10's refusal of `mutation` without `reach`, come with D10's later build.

Rationale: the writers of both a darn and a tapestry get the note from their own gate, and the review reads it.
An existing hub picks the defaults up with no reconcile: config load fills a template key that a tuned `gates` entry lacks, as `internal/gateslot/doc.go` states.

### Added after Discussion (arbiter, 2026-11-18): Mutation check order: trial before the mutation step (F6)

Decision: The board entry's order holds: the mutation trial comes before the `mutation` report step, and the step is built only if the trial pays. The record keeps D9's changed-line planner and D11's `lym gate mutate`, which are the hand replay the trial runs, built after failure triage. D10 stays designed in the record but is built only after the trial pays, in a later darn the parent opens from the trial's counts; no part of this task's first build builds it. The writer makes these edits: (1) Goal's build order reads: the reader and the reach note, then failure triage, then the mutation planner and `lym gate mutate` (D9, D11), then the trial, then the `mutation` step (D10) only if the trial pays. (2) D10 opens with a sentence saying it is built only after the trial pays, from the D11 board note's decision. (3) D11 prints the survivors, uncovered blocks and packages not judged that D9 finds, in the form D10 names, and its board note asks the parent, from the trial's counts, whether to build D10 and list `mutation` in the template's diff keys; the rationale sentence about shipping the step unlisted becomes one saying the step is not built before the trial says it pays. (4) D12 keeps `mutation_max_mutants` and `mutation_budget_min`, which D11 reads; the config check's acceptance of `mutation` as a report name and D10's refusal of `mutation` without `reach` move to D10's later build. (5) The D10 acceptance criteria are marked as checked at D10's later build. (6) The Open risks bullet 'Order of the mutation trial' is deleted, and the support log records this ruling.

Rationale: The board entry orders the mutation check as '1. Trial first' and '2. If the trial pays: a gate report step', and its study estimate is a trial darn, then 3 to 4 darns. The record establishes that the trial needs D9's planner and a hand verb, so those are the trial's tooling and stay; building D10 before the trial would go past the entry's condition, while dropping D9 to D11 would leave the mutation check undesigned, which the entry asks this task to design.

### Added after Discussion (arbiter, 2026-11-18): Reach turn bypasses Darn-Write's cold fixer (round 2 F1)

Decision: D3's reach-turn result goes to the writer alone, as the round-2 fix states in D3: gatefixer.WrapRowGate gains a check on the fixer entry's own gate closure, and on its forked closure when set, that spawns no cold fixer for a result whose findings are the reach note; every other verify failure still goes to the fixer first. The wrap marks its post-fixer re-run, and the gate counts no failure for a re-run, so the attempt-budget count is one per failing shuttle arrival. A reach note produced by the re-run after a fixer is returned to the writer as the re-run's verdict. This change to internal/gatefixer is inside the task's scope, since it is what makes the reach note reach the writer. Reviewers treat D3's bypass and its re-run mark as settled and do not reopen the channel choice.

Rationale: The board entry records the operator's wish (2026-11-18) that the writer get the note from its own verify gate, so it adds the missing tests itself; a cold fixer that receives the note and re-runs the gate to a pass would defeat that. The round-2 fixer applied exactly the fix round 2's F1 asked for, bound included, so the finding is answered in the record.

## Constraints

- `PATTERN-gate-slot`: every map refresh and mutant run holds a gate slot.
- `PATTERN-test-speed`: the background's "no gate builds a map" changes for D3's worktree maps; see `## PATTERN changes`.
- `PATTERN-fabric-boundary` and `PATTERN-gitrepo-client-boundary`: base contents, the parent branch and first-parent commits are read through `internal/fabricengine`.
- `PATTERN-lym-files`: the index is a cache beside the shared maps, named through `testcover.Dir`; its absence costs time and refuses nothing.
- `PATTERN-tolerant-records`: the report files and the index refuse no unknown key.
- `PATTERN-cli-cobra`: both verbs join the `gate` subtree.
- `PATTERN-stencil-ownership`: the rubric markers are filled by Go at fill time.
- `PATTERN-refusal-way-forward`: each refusal of D6 and D8 names its way forward, and a test reaches each.
- `PATTERN-code-conventions`: report names, file names and caps are named constants; the Go-only scope adds no new language branch.

## Auto-mode assumptions

- The caps of D4, D7 and D8 are named constants, not config keys: nobody asked to tune them.

## Open risks

- A large package's refresh in D3 can take minutes, since each tier 0 and tier 1 test runs alone; `reach_budget_min` bounds it.
- A writer that runs `lym gate test --gate batch` by hand on uncommitted edits gets a note from the maps there are, without its new functions;
  the note says so, names the dirty package, linked or its own, and tells it to commit and run again.

## Acceptance criteria

- D1: for a function reached by a test of its own package and by a test of another package, the reader returns both, the own package's first.
- D1: a worktree map whose file hashes match the worktree overrides the hub map for its package, and otherwise the later-built map is read.
- D1: an absent index, or one whose stamps differ, including by a map file added since it was written, gives the same answer as a current one, and only `lym gate test --map` run from the prime writes the index.
- D2: a new function and a function whose body changed are found; a function whose text is unchanged but whose lines moved, a function moved verbatim to another file of its package, a deleted function and a test file's function are not. A change to one of two build-constrained variants of a function is found in that variant's file alone.
- D3: a passing gate key listing `reach` writes `reach.json` and prints the note. A red tier, a call outside a hub and a non-Go scoper each give a report with its note and no lookup. The gate's exit is unchanged in every case.
- D3: a package whose worktree map matches the worktree is not refreshed. A package with an uncommitted file, one whose linked in-module package has one, one past `reach_budget_min` and every package of a call in the prime are answered from the existing maps, with a line saying which and why, naming a dirty linked package. No refresh writes the hub's shared maps.
- D3: a loomshed verify gate or weaver batch gate whose passing verify wrote a matching `reach.json` listing an unreached function no earlier turn listed returns one failing result carrying the note. A later arrival whose report lists only functions an earlier turn listed gets no turn, and neither does an arrival one failure short of the entry's attempt budget.
  With `darn.gate_fixer` set, a reach turn's result at Darn-Write spawns no cold fixer and reaches the Darn-Write writer's session.
- D4: the note lists own-package tests before other packages', fewer-functions tests first within each, at most five per function plus the count of the rest, and the map commits it read.
- D4: an unreached function whose package holds a tier 2 test or a spawn-marked entry carries the caveat. An unreached new function whose package other packages' older maps link carries a caveat naming those packages.
- D5: the `reach_report` note lists the report when it matches HEAD's tree. It says no current report exists, and why, when HEAD's changed functions, their hashes or the hashed Go files of a changed or named package differ, test files included, and when the report was written on a red tier, outside a hub or under another scoper.
- Review: the code review rubric tells the reviewer to read the reach note as evidence for "a behavior the diff adds with no coverage", and that an unreached function is a candidate, not a finding (D5).
- D6: `lym gate reach <file>#<Func>` prints the note for that function with no build. A function the maps do not hold gives a line saying so, with exit 0. A call with both a function and `--test`, or neither, or with `--since` or `--branch` but no `--test`, is refused naming both forms.
- D7: a failing gate's report gives a mapped failing test a first output line naming the changed functions it reaches, or "reaches none of this run's changes" followed by the D8 command on its own line, and an unmapped test "not in the coverage maps".
- D7: a failed subtest is looked up by its top-level test, and `gateslot.FailedTests` parses a report holding triage lines to the same failed tests as without them, a report at the line cap included, where triage lines are dropped before any test output is cut.
- D7: with no parent branch or an unreadable merge base, a mapped failing test's line says no change set exists and why, with no D8 command.
- D7: a failed lookup adds no line, and every fixer's report holds the triage lines of its failed tests.
- D8: `lym gate reach --test` lists, over the parent branch's first-parent commits after `--since`, each commit whose changed functions the test reaches, and names the map commit. `--branch` picks another branch.
- D8: a missing `--since` and a range over the cap are refused, each naming its way forward.
- D9: mutants land only on changed lines of covered blocks, at most one per line, in the operator order, under `mutation_max_mutants`. String constants, embedded lines and logger calls get none.
- D9: a changed uncovered block is reported uncovered with no mutant, and a package without a fresh map is reported not judged. In the prime, a package whose hub map matches the worktree is judged.
- D9: a mutant is killed by the first failing covering test, survives when every covering test passes, and is unjudged on a timeout or memory cap; a survivor lists other packages' reaching tests as possible killers.
- D10, checked at D10's later build: a gate key listing `mutation` writes `mutation.json` and its summary line, and the `mutation_report` note lists survivors, uncovered blocks and packages not judged only when the report matches HEAD's tree.
- D10, checked at D10's later build: a key listing `mutation` without `reach` is refused naming `reach`.
- D11: `lym gate mutate` refreshes the changed packages' worktree maps and prints D9's survivors, uncovered blocks and packages not judged for the worktree's diff, and `--base` changes the base it diffs from.
- Review: the run filed a board note asking for the trial and for the parent's decision, from its counts, on whether to build D10 and list `mutation` in the template's diff keys (D11).
- D12: a hub key without `reports` gets the template's `reports` with `reach` first in each key that lists `redundancy` and in `batch`, and the config check accepts `reach` and the three new keys.

## PATTERN changes

- `PATTERN-coverage-map-reader` — adds: Reading the coverage maps across packages or finding a diff's changed functions: only through `internal/testreach` (new); only the prime's `--map` run writes its index; a report step's review note is used only when it matches HEAD's tree. (test)
- `PATTERN-test-speed` — changes: Writing or running a test: via `lym gate test` by key; one `tiergate.Require` tier, lowest that fits; it pins behavior no other test covers, which the redundancy and reach reports check; skips, clocks, `t.Parallel`, `//lym:seam` and `//lym:watch` per the background. (test)

## Notes for the plan writer

- The `PATTERN-test-speed` background's coverage-map section needs its "built only on command" sentence changed for D3's worktree refresh, and a paragraph for the reach step.
