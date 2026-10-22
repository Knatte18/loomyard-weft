# Decision record — darn-verify-fixes

## Goal

A verify failure Darn-Write cannot own, one in code the task's diff cannot reach or one the parent has since fixed, no longer loops through Darn-Write.
A failure outside the task's impacted set is retried through Merge-In and then handed to the parent with a way forward that says it lies outside the task's impacted set;
a failure routed to Darn-Write while the parent has moved first takes the parent in through Merge-In;
an exhausted gate's findings file still exists when its halt is read;
and the impacted set stops adding every binary-driving test package to a diff that only reaches them through the `lyx` binary.

## Scope

In:

- Darn recipe's verify-failure routing at Merge-In and Publish: the failure record's target and the Stuck reason's way forward are chosen per failure from the impacted set (board items 3 and 4).
- Darn-Write's entry: a reroute to Merge-In when this run's failure record is present, the task branch has not moved since it, the task worktree is clean, and the parent has moved (item 3).
- The gate findings file of an exhausted gate, for every gated shed producer, outliving the shuttle run directory (item 5).
- `internal/impactset`'s rule for test packages that import `internal/testkit/lyxbin`, for every consumer of the impacted set (item 6).
- One added case in `TestRows_PatternsDenyAndAllow`; `internal/shellguard`'s rows are unchanged (item 2).
- The package docs these change: `internal/impactset`, `internal/landingshed`, `internal/loomshed`, `internal/shedadapters`, `internal/battenshed` where it describes the retry's target and the second halt's notice, and `contracts/recipes/darn-recipe.yaml`'s header comment.

Out:

- Item 1, `TwoWorktreesBootedConcurrently`: closed as fixed by `98b171b05`, with no reed change.
- The loom recipe's Publish failure routing: it keeps its fixed target `Webster-Burler-Review` and way forward.
- The Go driver's and batten's one-retry budget and their goto-and-resume mechanics, unchanged.
- Everything the board entry's re-scope lists as landed.

## Decisions

### D1 — A darn verify failure is classified as owned or not owned by the task

When Merge-In or Publish writes the verify failure record on a darn run, Go classifies the failure before writing the record's target and the Stuck reason.
The failure is **owned** when any of these holds:

- a failing test's package is in the impacted set `internal/impactset` derives from the task branch's merge base with its recorded parent to the working tree, the derivation Darn-Write's gate uses;
- a failing test is a `//lyx:guard` test the derivation lists in its guards;
- the derivation falls back to Whole, for any of its reasons;
- the failed run's log holds a package-level failure, a build failure or a crashed test binary, whose package is in the impacted set;
- the failed run's log yields only the opaque identity, which `parseVerifyFailures` in `internal/websterengine/verifyparse.go` returns only when it finds no test or package identity, or the record names no failing test;
- the derivation returns an error, which is logged at Warn.

Otherwise the failure is **not owned**.
The classifier reads the failure identities of the log the record names, every kind included, not only the record's tests, which hold test failures alone.
The record names a package by import path, and the impacted set by module directory and module-relative argument, guards by directory; the classifier maps both sides to one spelling, the import path, before comparing, nested modules included.
The classification runs in the CLI wiring, which resolves the worktree and merge base and hands `internal/landingshed` a classifier seam in place of the fixed `VerifyFailureTarget` and `VerifyFailedWayForward` values; a nil classifier keeps the fixed values, which the loom recipe keeps telling.

Rationale: the impacted set is what Darn-Write's own gate can run, so a failure outside it is one Darn-Write can neither reproduce in scope nor own.
Every uncertain case the parser reports counts as owned, so the classification only removes a Darn-Write trip whose failing package the import graph shows outside the diff's reach;
a test that drives the `lyx` binary can still reach the change (D5).
Bound: the classifier changes only the record's target and the reason's way-forward text; every failure still halts the row.

### D2 — An owned failure routes to Darn-Write as today; a not-owned one routes to Merge-In

An owned failure writes target `Darn-Write` and today's way forward.
A not-owned failure writes target `Merge-In`, and its Stuck reason names the failing packages, says they lie outside the task's impacted set, and gives the way forward in this order:
fix the failure on the parent branch if it fails there too, then run `lyx loom goto --to Merge-In` and `lyx loom resume` in the task worktree, which merges a moved parent and re-runs the failing gate;
or, if the parent judges the task did cause it, run `lyx loom goto --to Darn-Write` and `lyx loom resume`, where a moved parent is merged in first (D3).
For a failing package whose tests import `internal/testkit/lyxbin`, the reason also says the test drives the `lyx` binary and may reach the task's change, so the Darn-Write goto is the way forward when the test does not fail on the parent.

The batten retry and the Go driver's verify retry read the target from the record, so their one automatic retry of a not-owned failure becomes a Merge-In re-entry, which `lyx loom goto` admits from a halt at Merge-In itself.
Merge-In takes the parent lock and merges a moved parent;
the gate that failed then re-runs at its own row, since it has no recorded pass on that tree: Merge-In's `landing` call for a Merge-In failure, Publish's `publish` call for a `publish` gate failure, whose tree Merge-In and Publish's `landing` call skip.
That re-run also gives a flaky test one more run.
A second failure finds the retry spent and halts for the parent with the reason above, at Merge-In or Publish, through the existing blocked notice.

Rationale: the two things that clear a failure outside the task's reach are a fix on the parent and a re-run of a flake; Merge-In does the first under the parent lock (`PATTERN-parent-lock`) and the verified-tree record makes the failed gate run again, and the writer can do neither.
Bound: the edge only redirects the existing one retry; it adds no retry, and a not-owned failure reaches the parent after at most one Merge-In re-entry.
It relies on `internal/verifytree` re-running a command on a tree with no recorded pass, which a failed run never writes.

### D3 — Darn-Write reroutes to Merge-In when this run's failure record is present, the branch has not moved since it, the tree is clean, and the parent has moved

At the start of each Darn-Write call, before the writer session is spawned, the producer checks four facts through told seams:

- a verify failure record is present and was written during this run, its `written_at` at or after the run's start;
- the task's HEAD is the record's `head`, so no commit has landed on the task branch since the failure;
- the task worktree has no uncommitted changes, read through a told seam over the dirty-paths reader Merge-In's clean check uses, so the two never disagree, and a failed status read counts as dirty, so the writer runs;
- a tip Merge-In would merge is not an ancestor of HEAD, read with no fetch: the local parent branch alone when the run skips pushes, and the local parent branch and its existing upstream tracking ref otherwise.

When all four hold, the row returns Stuck with Reroute set, and the darn recipe gives `Darn-Write` an `on_reroute: Merge-In` edge, so no writer is spawned and no bounce budget is spent.
The reroute's reason carries no resume way forward; it says the parent moved since the failure and Merge-In takes it in, and the prior-work note's reader of the latest Darn-Write outcome skips a reroute entry, so a later writer spawn still names the last real halt.
Merge-In then merges the parent under its lock, verifies the merged tree, and on a pass hands to Publish as usual.
A tip read that fails logs a Warn and counts as not moved, so the writer runs as today.
The check shares landingshed's tip-reading seam, not its moved predicate, which compares against the merge record.
The way forward of a gate-exhausted Darn-Write halt says the resume re-spawns Darn-Write, or first takes a moved parent in through Merge-In, so the text matches what the resume does.

Rationale: an owned failure routed to Darn-Write may be one the parent has fixed since; the writer's gate runs on the task branch, which takes the parent only at Merge-In, so without the merge the writer loops on a fixed failure.
Merging belongs to Merge-In alone (`PATTERN-parent-lock`), so the producer reroutes instead of merging itself.
Bound: the reroute skips the Darn-Review perch and Describe, and the HEAD and clean-tree conditions make that safe: the branch carries no commit and the worktree no uncommitted edit since the failed verify, which ran on reviewed code, so the only new code is the parent merge, the skip a moved parent already takes from Publish.
A writer that committed after the failure disables the reroute through the HEAD condition, one that left uncommitted edits disables it through the clean-tree condition, and in both cases the writer runs.
A passing Merge-In or Publish removes the record.
After Merge-In merges, the parent tip is an ancestor of HEAD, so the next Darn-Write entry does not reroute again.
When Merge-In is Stuck without merging, on a parent pair that is dirty, mid-merge, ahead of or diverged from its upstream, a later Darn-Write entry reroutes back to it, and once Darn-Write's reroutes since its latest Done or reroute-cap halt reach shed's reroute cap, the next one halts it.
That halt embeds the reroute's reason, which points the parent at Merge-In's Stuck way forward, repairing the parent pair, before the resume the cap halt names.

### D4 — An exhausted gate's findings survive the shuttle run directory

When a gated shed producer's gate is exhausted, the last failing entry's findings text is written to a per-row file under the run's never-tracked shed run directory, whose path `internal/shedrun` declares (`PATTERN-shed-run-directory`), and the Stuck reason names that file instead of the shuttle run directory's `gate-findings.md`.
The file is overwritten by the row's next exhausted gate and removed by the row's next call whose gates pass, so rows that never return Done, the Burler review and fix rows, clear it too.
`lyx loom goto`, `lyx loom resume` and a pause leave it in place.
`internal/shuttleengine` hands the findings text out on its gate outcome, since its finalize removes the run directory before the producer reads anything.
Each gated producer in `internal/shedadapters` is told its row's findings-file path, writes the file through the shared reason helper when its gate is exhausted, and removes it on a call whose gate outcome passed, so every gated producer gets it, Darn-Write's writer included.

Rationale: the halt reason is read after the shuttle run has finalized, so a path inside the run directory is always gone by then.
A terminal entry's reason already rides the outcome as text and keeps doing so.

### D5 — The impacted set adds binary-driving test packages only for a change to `cmd/lyx` itself

The rule "when a changed package is a dependency of `cmd/lyx`, every package whose tests import `internal/testkit/lyxbin` joins the set" is replaced by: every such package joins when the diff changes the `cmd/lyx` main package itself.
Every other package reaches the set only through the import graph, so a binary-driving test package joins when its own test closure reaches a changed package.
The change applies to every consumer of the impacted set: darn's gates, loom's round gate and every computed-scope `lyx gate test`.

Rationale: almost every package of the module is a dependency of `cmd/lyx`, so the old rule added every binary-driving test package to almost every diff; that is how a change confined to `internal/planglyph` ran `internal/reedcli` (#785).
The `cmd/lyx` main package is imported by no test, so the graph cannot see a change to it, and that change alone keeps the blanket join.
Bound: a binary-driving test that reaches a change only through a subcommand its own package does not import leaves the narrowed run; the whole-tree runs at Merge-In, Publish and Finalize still run it, as the package doc's bound states, and D1 counts such a failure as not owned.
A darn diff the hub's tree scope covers runs the tree-scoped command instead, whose reach the hub's `darn.yaml` sets.

### D6 — The shell guard pins the darn writer's tmux-tier reproduce command

`TestRows_PatternsDenyAndAllow` gains one case, and `internal/shellguard`'s rows are unchanged: the reproduce form the round gate renders for a failing tmux-tier test, `lyx gate test --wait --tier 2 github.com/Knatte18/loomyard/internal/x -- -run '^TestX$'`, passes every row.

Rationale: a darn session can run the failing tier through `lyx gate test --tier 2`, which no row refuses, and the Darn-Write round gate already re-runs the record's tests itself; the case keeps a future guard pattern from silently taking that away.

## Constraints

- `PATTERN-parent-lock`: only Merge-In merges the parent into the task branch; D2 and D3 reach the merge only by routing to Merge-In.
- `PATTERN-told-geometry` and `PATTERN-cliwire-sole-wiring`: the classifier (D1), the tip reader and the status reader (D3) are built in the CLI wiring and told to `internal/landingshed` and `internal/loomshed` as seams.
- `PATTERN-fabric-git`: D3's tip read goes through fabricengine's parent-tips reader, and its status read reuses the dirty-paths reader Merge-In's clean check uses, a read-only `status --porcelain`, which `PATTERN-fabric-git` exempts.
- `PATTERN-shed-run-directory` and `PATTERN-durable-vs-ephemeral-state`: D4's file path is declared in `internal/shedrun` alone, never tracked, under `.lyx`.
- `PATTERN-refusal-way-forward` and `PATTERN-no-denied-recovery`: D2's not-owned reason names its way forward with commands the agents' settings allow, and a test reaches it.
- `PATTERN-shuttle-provider-seam`: D4's change to `internal/shuttleengine` touches the gate outcome only.
- `fabric-remote-async` (board, operator decision 2026-10-18): no lyx command waits on the network, so D3 reads tips without a fetch.

## Auto-mode assumptions

- Item 1 is closed against the re-scope's open list, on the root cause `98b171b05` removed, a commit the re-scope's list of landed commits does not name.
- D3 reads tips without a fetch, accepting that a parent fix seen only on origin is missed until something fetches.

## Open risks

- A D3 reroute whose Merge-In verify still fails, an owned failure the parent's change did not cure, finds the one retry spent by the goto that brought the run to Darn-Write, and halts for the parent instead of reaching the writer.
- An owned failure the parent fixes after the writer has committed is not rerouted (D3's HEAD condition); the writer loops through its gate budget and the run halts at Darn-Write with no in-run way forward that keeps the writer's commits.
  `gotoAdmitted` in `internal/shedengine/goto.go` admits from that halt only Preflight and Darn-Write, and goto's refusal says it never skips a review row, so Merge-In is out of reach, and a resumed writer's gate cannot pass without the parent merge.
  The parent's recovery drops the writer's work: reset the task branch to the record's `head` and discard every uncommitted change, untracked files included, since D3's clean-tree condition counts them, then `lyx loom goto --to Darn-Write` and `lyx loom resume`, where D3's entry check reroutes to Merge-In.
- An owned failure the parent fixes while the writer has left uncommitted edits is not rerouted either (D3's clean-tree condition); the writer loops through its gate budget and the run halts at Darn-Write the same way.
  The parent's recovery discards the edits, since committing them would trip D3's HEAD condition instead, then `lyx loom goto --to Darn-Write` and `lyx loom resume`, where D3's entry check reroutes to Merge-In.
- An unparsed failure beside a parsed failure outside the impacted set classifies as not owned, since the parser emits no opaque identity once it has found another (D1); the cost is bounded by D2's one Merge-In re-entry and the parent hand-back, whose reason then wrongly places the failure outside the task.
- A deterministic not-owned failure on an unmoved parent spends one futile gate re-run before the hand-back, the cost `finalize-dirty-names-paths` and `gate-test-repeats` warn about; D2 keeps it for the flake case.

<!-- lyx:carry-over Discussion-Review -->
- Discussion-Review, round 2: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lyx/reviews/discussion/round-2-review.md; fixer report: _lyx/reviews/discussion/round-2-fixer-report.md.
  - open-risks-name-refused-goto (class design, severity MEDIUM, open in rounds 2)
<!-- /lyx:carry-over Discussion-Review -->

## Acceptance criteria

- A darn Publish failure whose failing tests all lie in packages outside the impacted set writes target `Merge-In`, and its Stuck reason names those packages and the Merge-In way forward.
- A darn Publish or Merge-In failure with a failing test in the impacted set, a failing guard test, a Whole fallback, no failing tests or a derivation error writes target `Darn-Write` with today's way forward.
- A darn failure whose log holds a build failure in an impacted package and a failing test outside the set writes target `Darn-Write`; one whose log yields only the opaque identity writes target `Darn-Write`; one whose only failures, package-level and test failures alike, lie in packages outside the set writes target `Merge-In`.
- A not-owned reason for a failing package whose tests import `internal/testkit/lyxbin` says the test drives the `lyx` binary and names the Darn-Write goto for a test that does not fail on the parent.
- The owned check matches a record's import-path package against an impacted set listed by module-relative argument.
- A loom recipe failure writes target `Webster-Burler-Review` as today.
- The batten retry of a not-owned failure sends the child to Merge-In and resumes it; a second failure halts with the retry spent and the not-owned reason in the blocked notice, at Merge-In and at Publish.
- A Darn-Write call with this run's failure record, HEAD at the record's `head`, a clean task worktree and a parent tip outside HEAD returns a reroute to Merge-In without spawning the writer, without spending bounce budget and without the resume way forward.
- A Darn-Write call with no record, a record from before the run's start, a HEAD past the record's `head`, a dirty task worktree, a failed status read, a parent tip already in HEAD, an upstream tip outside HEAD on a run that skips pushes, or a failed tip read spawns the writer as today, and its prior-work note names the last non-reroute outcome.
- A gate-exhausted Darn-Write halt's way forward names both the Darn-Write re-spawn and the Merge-In reroute a moved parent takes.
- After a gated producer's gate is exhausted, the file its Stuck reason names exists and holds the last failing findings, also after `lyx loom goto` and `lyx loom resume`; the row's next call whose gates pass removes it.
- The impacted set of a diff confined to `internal/planglyph` holds no package that reaches `internal/planglyph` only through the `lyx` binary, `internal/reedcli` included.
- The impacted set of a diff changing the `cmd/lyx` main package holds every package whose tests import `internal/testkit/lyxbin`.
- The shell guard lets `lyx gate test --wait --tier 2 github.com/Knatte18/loomyard/internal/x -- -run '^TestX$'` through every row.

## Notes for the plan writer

- `GateOutcome.FindingsPath`'s doc in `internal/shuttleengine/gate.go` states why the path is dead once `Wait` returns; D4 replaces that contract.
- `machine-handover` runs concurrently and changes `internal/landingshed`'s Merge-In and Finalize; expect merge conflicts in its deps and docs.
