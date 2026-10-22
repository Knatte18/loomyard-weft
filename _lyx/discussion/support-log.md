# Support log — darn-verify-fixes

## Interview

Autonomous run: no operator answers; every pick below is the writer's own, recorded in the question ledger.

### Turn 1 — exploration

- Read the board entry (re-scoped 2026-10-22) and the related entries `gate-test-repeats`, `gate-test-upto`, `tender-pause-holds-reprompt`, `card-halt-is-question` and `loom-gates-xdr-stacking`; none plans the routing, the findings file or the impacted-set rule.
  `gate-test-repeats` notes that tmux tests have been the flaky ones in gates, citing this task; nothing here changes which gate runs tmux.
- Traced the route a Publish failure takes: landingshed writes the verifytree failure record with the recipe's fixed target, and batten's Run-Shed retry or the Go driver's verify rule sends the child there once through `lyx loom goto` and resumes it.
  The darn target is fixed at `Darn-Write`, whose round gate re-runs the record's failing tests through `lyx gate test --wait --tier N <pkg> -- -run`.
- Item 1: `98b171b05` (2026-10-18) moved the capability probe's `list-commands` to a private `-L` socket, which is the exact failure #714 (2026-10-17) reports. 40 of 40 runs of `TestBoot_TmuxConf` passed through `lyx gate test --wait --tier 2`.
- Item 2: no shell guard row refuses `lyx gate test --tier 2`; this session ran it.
- Item 5: shuttle's finalize removes the run directory, `gate-findings.md` included, on the Done cleanup an exhausted gate takes, and `gateFailedReason` names that path afterwards.
- Item 6: `planglyph` is in `cmd/lyx`'s dependency closure, and `internal/reedcli`'s smoke tests import `internal/testkit/lyxbin`, so the blanket lyxbin rule selected it.
  Ten test packages import `lyxbin`, and they drive several subcommands each, not only their own.

### Turn 2 — design picks

- Classification by impacted set (D1), conservative on every uncertain case.
- Not-owned target is Merge-In (D2), so the existing single retry merges a moved parent and re-runs a flake.
- Item 3 as a Darn-Write entry reroute to Merge-In (D3), keeping merges in Merge-In under the parent lock.
- Findings file kept per row under the shed run directory until the row's next call whose gates pass (D4).
- lyxbin join only for a change to the `cmd/lyx` main package (D5).
- One pinning test case for item 2 (D6); no reed change for item 1.
- The board's 2026-10-20 case, a failing test outside the branch, is not owned under D1 and never reaches Darn-Write.

### Turn 3 — lens review

Every lens returned a report.

- bound-breaker, BLOCKING, D3 reroute skips the perch over writer commits made after the failure: fixed, D3 requires HEAD at the record's `head`.
- bound-breaker, MEDIUM, stale record from an earlier run reroutes before any writer: fixed, D3 requires the record written during this run.
- bound-breaker and walkthrough, MEDIUM/LOW, D1 compares import paths against module-relative arguments: fixed, D1 maps both to import paths; acceptance criterion added.
- bound-breaker, MEDIUM, D3 "cannot cycle" false while Merge-In is Stuck on a broken parent pair: fixed, D3's bound names that path.
- bound-breaker, interactions, walkthrough and leanness, LOW, D2 claims Merge-In re-runs `landing` for a `publish` gate failure: fixed, D2 says the failed gate re-runs at its own row.
- bound-breaker and existing-paths, LOW/MEDIUM, D4's "next Done" never fires for the Burler rows: fixed, removal on the row's next call whose gates pass.
- bound-breaker, LOW, D5 bound ignores tree-scoped diffs: fixed, D5 names the tree-scoped command's reach.
- bound-breaker, LOW, 40 runs in one binary do not reproduce the original load: accepted, the record now rests item 1 on the root cause, not the run count.
- goal-scope, MEDIUM, item 1 closed against the re-scope without saying so: fixed, auto-mode assumption added.
- goal-scope and walkthrough, MEDIUM/LOW, D3 checks only at a call's start, so a mid-session parent fix still costs the gate budget: fixed as stated bounds, Turn 2 notes the 2026-10-20 case is not owned under D1, and Open risks names the residual.
- goal-scope, LOW, Q9 already answered by `gotoAdmitted`: fixed, D2 states goto admits the halted row; risk removed, Q9 resolved.
- goal-scope, LOW, D6 adds a test the board did not ask for: rejected, a one-row pin of the behaviour item 2 relies on.
- board-fit, MEDIUM, D3's fetch contradicts `fabric-remote-async`: fixed, D3 reads tips with no fetch; constraint and auto-mode assumption added.
- board-fit, LOW, new way-forward texts tie to `run-verbs-shared`: rejected, that entry already covers every `lyx loom` way-forward text.
- board-fit, LOW, futile re-run cost against `finalize-dirty-names-paths` and `gate-test-repeats`: fixed, Open risks names it.
- board-fit, LOW, `machine-handover` touches landingshed concurrently: fixed, Notes for the plan writer.
- board-fit, LOW, D2 relies on a failed tree always re-running: fixed, D2's bound states it.
- existing-paths, LOW, reroute history entry misleads the prior-work note and gains the resume way forward: fixed in D3 and acceptance.
- existing-paths, LOW, D5 applies beyond darn: fixed, D5 says every consumer.
- existing-paths, LOW, retry-spent notice wording keyed on Publish: fixed, acceptance covers Merge-In and Publish halts; battenshed doc in Scope.
- interactions, MEDIUM, D3 redirects the parent's explicit goto to Darn-Write: fixed, D2's way forward says a moved parent is merged first.
- walkthrough, MEDIUM, a D3 reroute that does not cure the failure costs a parent turn: fixed, Open risks.
- walkthrough, LOW, D3 says it shares landingshed's moved predicate: fixed, D3 shares only the tip reader.
- leanness, MEDIUM, Auto-mode assumptions repeat Decisions: fixed, cut to the two picks not stated elsewhere.
- leanness, MEDIUM, item 1 evidence belongs here: fixed, Scope keeps the conclusion only.
- leanness, LOW, duplicate Open risk of D5's bound, D5's package list, item 2 stated twice, Notes repeating `relevant.md`, generic Constraints: fixed, each cut.

## Rejected alternatives

- **Darn-Write merges the parent itself** (item 3): duplicates Merge-In's lock, conflict handling and origin reconcile, and breaks `PATTERN-parent-lock`.
- **The writer runs `lyx fabric merge-in` when told** (the orch's 2026-10-20 workaround): a merge outside the parent lock.
- **Decide the moved parent at the retry's goto instead of at Darn-Write's entry**: the retry fires right after the failure, before a parent fix usually lands; the 2026-10-20 case had the fix land while Darn-Write looped.
- **A not-owned failure halts straight to the parent with no target**: loses the one automatic re-run, which is what clears a flake or picks up a parent fix without a parent turn.
- **Classify by changed files only (failing package changed or not)**: misses a failure in an untouched package that imports a changed one, which the task can cause.
- **Keep the lyxbin rule and annotate each binary-driving test package with the subcommands it drives**: precise, but a new per-package annotation and a new invariant to keep in sync for ten packages, against a bound the whole-tree sites already cover.
- **Drop the lyxbin rule entirely**: a change to the `cmd/lyx` main package, which no test imports, would then select no binary-driving test.
- **Make `TwoWorktreesBootedConcurrently` derive probe socket keys from pid and a counter**: two engines in one test process share a pid, and the random 28-bit key's collision chance does not justify it; the reported failure mode is already gone.
- **D3 fetches the parent's upstream before reading tips**: contradicts the `fabric-remote-async` operator decision that no lyx command waits on the network.
- **D3 reroutes whatever the task branch did since the failure**: ships writer commits past the Darn-Review perch and Describe.
- **Darn-Write's round gate ends the session for a reroute when the parent moves mid-loop**: the session's committed edits would then skip review, the same flaw.
- **Extend the classification to the loom recipe**: not asked by the board entry; loom's target is a review round, not a looping writer.
- **Keep the whole shuttle run directory on an exhausted gate**: leaves the strand's other run files behind for a single findings file.

## Review rounds

- Round 1, BLOCKING: D3 gains a clean-tree condition and reads only the tips Merge-In would merge, and its Stuck-pair bound names the reroute cap;
  D1 counts a log with an opaque failure, or a package-level failure in the set, as owned;
  D1, D2 and the Goal stop calling a not-owned failure unreachable, and D2's reason flags binary-driving packages;
  D4 names the producers as the file's writers and removers;
  D6 pins a test case, not a guard row, in the import-path form.
- Round 2, APPROVED: the Open risks for a writer that committed or left edits stop naming a goto to Merge-In, which `gotoAdmitted` refuses from a Darn-Write halt, and name the parent's recovery, dropping the writer's work so D3 reroutes;
  D1's opaque rule follows the parser, which emits the opaque identity only when it finds no other, and Open risks names the mixed-log residual;
  the `PATTERN-fabric-git` constraint names the parent-tips reader and Merge-In's dirty-paths reader.

## Question ledger

- Q1 (resolved, auto-pick): Is item 1 still open? — No; `98b171b05` fixed the shared-socket probe race and 40 of 40 runs pass. No reed change.
- Q2 (resolved, auto-pick): Can a darn session run the failing tier? — Yes, through `lyx gate test --tier 2`; pin it with one case in the guard's rows test (D6).
- Q3 (resolved, auto-pick): How does Go tell code the task touched from code it did not? — By the impacted set from the merge base, with guards, Whole fallback, package-level failures in the set, a log yielding only the opaque identity, empty test lists and errors counted as owned (D1).
- Q4 (resolved, auto-pick): Where does a not-owned failure go? — Merge-In, through the existing one retry, then the parent (D2).
- Q5 (resolved, auto-pick): Where does the parent merge of item 3 happen? — Merge-In, reached by a Darn-Write entry reroute when a failure record is present and the parent moved (D3).
- Q6 (resolved, auto-pick): Where does the findings file live and when is it cleared? — Per row under the shed run directory, removed on the row's next call whose gates pass (D4).
- Q7 (resolved, auto-pick): How is the lyxbin rule narrowed? — Join only for a change to the `cmd/lyx` main package; everything else through the import graph (D5).
- Q8 (resolved, auto-pick): Does the classification apply to loom? — No; darn only.
- Q9 (resolved, lens evidence): Does `lyx loom goto` accept the row the run is halted at? — Yes; `gotoAdmitted` in `internal/shedengine/goto.go` admits the reference row for paused, blocked and failed runs.
- Q10 (resolved, auto-pick): Item 1 departs from the 2026-10-22 re-scope's open list — closed on the root cause `98b171b05` removed, stated in the auto-mode assumptions for the parent review to overrule; the re-scope's list of landed commits does not name `98b171b05`.
- Q11 (resolved, auto-pick): Does D3 fetch the parent's upstream? — No, per `fabric-remote-async`; it reads the tips Merge-In would merge: the local branch alone under push skip, and the local branch and its tracking ref otherwise.
- Q12 (resolved, auto-pick): How is D3 kept from skipping review of writer commits or acting on a stale record? — HEAD must equal the record's `head`, the task worktree must be clean, and the record must be written during this run.
- Environment note: `lyx loom validate-discussion` and the tier-2 gate run worked; nothing looked broken.
