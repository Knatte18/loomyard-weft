---
format: 5
approved: false
language: go
---

# Plan: darn-verify-fixes

A darn verify failure Darn-Write cannot own no longer loops through Darn-Write.
The impacted set stops adding every binary-driving test package to a diff that reaches them only through the `lyx` binary (D5); Merge-In and Publish classify each darn failure against that set and route a not-owned one to Merge-In with a reason saying so (D1, D2), which batten's one retry follows; Darn-Write reroutes to Merge-In when the parent has moved since this run's failure on an unchanged branch and tree (D3); an exhausted gate's findings outlive the shuttle run directory (D4); and the shell guard pins the darn writer's tmux-tier reproduce command (D6).

## Card Index

1 — impactset-cmd-lyx-join — join binary-driving test packages only for a change to `cmd/lyx` itself, and list them on `Targets`
2 — landingshed-failure-route — Merge-In and Publish take the record's target and the reason's way forward from a told classifier
3 — darn-failure-classifier — classify a darn failure as owned or not owned against the impacted set and route a not-owned one to Merge-In
4 — batten-retry-merge-in-target — pin batten's retry of a Merge-In target and the not-owned reason in the second halt's notice
5 — shuttle-gate-findings-text — carry the failing entry's findings text on the gate outcome
6 — shedrun-gate-findings-file — declare the per-row gate findings file under the run's scratch directory
7 — shedadapters-keep-gate-findings — gated producers write, name and clear the told findings file
8 — wire-gate-findings-files — tell every gated row its findings file
9 — darn-write-reroute-edge — give Darn-Write an `on_reroute: Merge-In` edge and skip reroutes in the prior-work note
10 — darn-write-entry-reroute — Darn-Write reroutes to Merge-In on this run's failure, an unmoved branch, a clean tree and a moved parent
11 — shellguard-gate-test-case — the shell guard lets the tmux-tier reproduce command through every row

## Shared Decisions

- The classifier lives in `internal/loomcli`, where the worktree and merge base are resolved (`PATTERN-told-geometry`, `PATTERN-cliwire-sole-wiring`); `internal/landingshed` sees only the `FailureRoute` it answers.
- A not-owned reason leads with the packages before the merge clause, because batten's blocked notice keeps only the first 200 characters of the child's error.
- Darn-Write's reroute seams are told by `internal/loomcli`, since `internal/loomshed` may not import `internal/fabricengine`.
