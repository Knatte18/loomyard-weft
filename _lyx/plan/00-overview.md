---
format: 5
approved: false
language: go
---

# Plan: origin-reconcile

A machine must never act on a stale view of origin, and a push origin refuses must never be silent.
The plan builds bottom-up: the failed-push record and its notice route, the board seam and the records replay, the pull and push paths on that replay, the local removal mode, the gate exemption, then the origin reconcile in three steps (judge, move, ended entries), its seams, and last its five call sites.

## Card Index

1 — failed-push-record — the failed-push record store, its once-per-episode notice through an installed notifier, and its way forward per class
2 — push-notifier-install — orchcli installs the FABRIC notice route in every lyx process from the root pre-run
3 — board-replay-seam — fabric's board seam and boardengine's by-entry Bolt replay, installed from the root pre-run
4 — records-replay — the weft and Bolt records replay and D3's ended-set rule
5 — bolt-pullfirst-replay — Bolt's pull-first replays instead of dropping seed commits
6 — pull-replay-reflog — `Fabric.Pull` replays the weft or fails with a way forward, and judges warp by the reflog
7 — land-weft-stop — Land deploys after a weft stop and names it in its notice
8 — push-auth-class — gitrepo returns an auth refusal of a push as its own class
9 — push-retry-and-record — every fabric push classifies, retries records rejections by replay, and records and reports a failure
10 — failed-push-printing — fabric status, tender status and cleanup print outstanding failed pushes; cleanup stops reading a spawn as landed
11 — preflight-detached-push — the Preflight row's board push goes detached
12 — gate-entry-exemption — the network gate may pass one named pause entry for the process that set it
13 — remove-local-mode — fabricengine's local removal and the uncommitted-path snapshot
14 — pairteardown-local-mode — pairteardown's local mode, snapshot and no board write
15 — origin-reconcile-judge — the origin reconcile's types, network calls, branch judgment and D7 classifier
16 — origin-reconcile-moves — the moving reconcile: fast-forwards, replays, locks, live and busy
17 — origin-reconcile-ended — ended entries torn down through the seam, and stale failed-push records cleared
18 — reconcile-seams — pairteardown builds the reconcile's live-pair, parent-lock and teardown seams
19 — orch-start-reconcile — `lyx orch start` reconciles the hub first and reports, never refusing
20 — cleanup-reconcile — `lyx cleanup` reconciles first, judge-only without `--apply`
21 — batten-resume-reconcile — batten resume runs the run-scope reconcile
22 — landing-reconcile — Merge-In and Finalize run the run-scope reconcile
23 — wake-reconcile — the hibernate wake reconciles before the lift

## Shared Decisions

- **Process-global installs for the push path's seams.** D1 says `fabriccli` wires the push path's seams for the detached push child. `fabriccli` cannot import `orchcli`: `orchcli`'s in-package tests import `hubforge`, which imports `fabriccli`, so the import closes a test-build cycle. The notice route (card 2) and the board replay (card 3) are therefore installed process-wide from `cmd/lyx`'s root pre-run, as the network gate is, which covers the detached push child and every other pushing process alike. The seams the reconcile itself takes (live pair, parent lock, teardown) stay injected by each call site, built in one place, `pairteardown.Teardown.OriginSeams` (card 18).
- **Process-global state in tests.** Test binaries never run the root pre-run. A test reaching a Bolt replay or the ended-set rule calls `boardkit.InstallReplay` (card 3), a once-per-process install of the same value; a test installing a push notifier, or swapping a package-level test seam through a `ForTest` setter, runs without `t.Parallel`, and the notifier carries the hub path so a fake counts only its own hub's calls.
- **Transport failures in tests** are reached with an origin URL of `http://127.0.0.1:1/x`, which git refuses with `connection refused`, a phrase `gitexec.IsTransportFailure` matches; a missing local path is not a transport failure.
- **The Preflight row's synchronous records push** is `hubreconcile`'s `reconcileHubWide` `bolt.Push`, the only synchronous push of the records repository the Preflight row runs (through `hubreconcile.Ensure`); `Fabric.Commit` and the status seam already spawn theirs detached. Card 11 makes it the detached board sync.
- **Reflog reading.** D4's rewound and rewritten tests and D9's unpushed test share `trackingReflog` and `classifyRewrite` (card 6), which the reconcile cards reuse.
- **hibernateengine stays free of fabricengine.** Its `Deps.Reconcile` returns the engine's own `WakeReconcile` view, which hibernatecli maps from the report.
- **Failed-push record identity** is repo side plus branch, one file each under `<board>/.lyx/failed-push/`.

## PATTERN changes

- `PATTERN-origin-reconcile` — adds: Acting on a run or the hub after this machine may have been away (orch start, wake, batten resume, Merge-In, Finalize, cleanup): first `fabricengine`'s origin reconcile, the sole judge of local branches against origin.
- `PATTERN-push-failure-visible` — adds: A push origin still refuses after its retry: a failed-push record and one creel notice to the orch, never only a log line. (test)

## verify:

```
go build ./cmd/lyx
lyx gate test --tags integration ./cmd/lyx ./internal/lyxdirs ./internal/fabricengine ./internal/orchcli ./internal/boardengine ./internal/hubreconcile ./internal/fabriccli ./internal/battenshed ./internal/battencli ./internal/gitrepo ./internal/tendercli ./internal/cleanupcli ./internal/pauserecord ./internal/pairteardown ./internal/landingshed ./internal/loomcli ./internal/hibernateengine ./internal/hibernatecli ./internal/pattern ./internal/testkit/boardkit
```
