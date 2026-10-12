---
format: 5
approved: false
---

# Plan: merge-review

A run that merges its parent in gets a cold-start Merge-Review of how its code interacts with what the parent gained, and no other run can move that parent until the run lands or stops making progress.
The plan builds the parent lock package, the engine's reroute edge and the loop's hooks first, then the landing pieces (the merge record, the shared guard and the Merge-In producer), then the Bouncer and recipe seams, then flips both recipes to put Merge-In ahead of Publish.
After the flip it makes Publish, PR-Gate and Finalize hold the lock and reroute on a moved parent, wires the loop's and batten's lock handling, adds the status note and the lock verbs, and updates the orch stencil.

## Card Index

1 — parentlock-store — the per-branch parent lock record, ticket queue, compare-and-release and force releases
2 — parentlock-wait-idle — the queue wait, the wait record and the idle rule
3 — shedengine-reroute — the on_reroute edge with its three-reroute cap, and a halting Stuck
4 — shedbuild-on-reroute — the on_reroute recipe key and the cap's told way forward
5 — shedverbs-loop-hooks — the loop's exit and wake hooks
6 — shuttle-agent-waits — one reading of each live run's waiting and busy state
7 — fabric-parent-reads — parent tips with their relation, changed paths, the holds-work test and the pair fast-forward
8 — verify-failure-row — the failure record names the row that ran the verify
9 — landing-idle-config — landing.yaml's parent_lock_idle_release_min
10 — landing-merge-record — the merge record, its overlap, the skip decision and the review note
11 — landing-parent-guard — the lock and parent-tip seams in landing's Deps, and the shared guard
12 — landing-merge-in-producer — the Merge-In producer
13 — segment-colour-merge — a merge segment colour for the review's sessions
14 — bouncer-blocking-and-merge-note — BLOCKING-aware auto-accept and the merge_review rubric marker
15 — loom-merge-config — loom.yaml's merge perch keys
16 — recipe-merge-seams — the Bouncer and Burler entries' merge keys and their wired seams
17 — merge-in-row — Merge-In and the Merge-Review perch in the recipes; Publish loses its merge-in
18 — merge-review-off-switch — skip_merge_review routes Merge-In straight to Publish
19 — guarded-landing-rows — Publish, PR-Gate and Finalize hold the lock and reroute on a moved parent
20 — loop-lock-hooks — the loop releases or hands off its lock at exit and on an idle bound
21 — status-lock-wait — a lock wait reads as waiting in lyx loom status
22 — batten-retry-rows — batten's verify retry follows the failing row; a lock wait is a live wait
23 — batten-lock-backstop — batten holds a PR-Gate lock, backstops a stalled loop and resumes on a moved base
24 — batten-lock-verbs — lyx batten lock list and lock release
25 — orch-stencil-pr-gate — the orch's PR-Gate fork judges goal and decisions

## Shared Decisions

- **Lock identity.** A lock owner is a run (slug, pair path, run id) plus a process (pid, start time). The loop owns the lock when its pid record names a live loop, the step process otherwise, and batten's process while it holds a handed record. Every release and hand-off compares on the process, so a newer owner's record is never freed.
- **Where the lock lives.** `<BoardDir>/.lyx/parent-lock/`, built by `parentlock.Dir` through `lyxdirs` and told to every engine; only CLI wiring resolves it (`PATTERN-told-geometry`, `PATTERN-hub-containment`).
- **Lock Stucks halt in place.** A lock wait paused by the operator, and an unreadable lock record, return Stuck with `OutputPointer.Halt`, so PR-Gate, the one landing row with an `on_stuck` route, halts at its row instead of bouncing to rework.
- **Moved parent.** Moved means a code-side tip (local, or upstream after a fetch unless the hub skips pushes or the branch has none) differs from the merge record's `merged_tip` and does not hold the task's work by `HoldsWork`. A missing merge record counts as moved, so the row reroutes to Merge-In, which writes one.
- **The perch's models** are loom.yaml's Webster-Review keys, as darn's perch already takes them: both review a committed diff.
- **Run records stay ephemeral.** The merge record, the reviewed-tip file and the wait record live in the run's `.lyx` scratch directory and are never committed.

## Rename mechanic

1. Run `git mv <old> <new>` FIRST, before any other change to the moved file.
2. Then make ONLY surgical edits (package declaration, imports, identifier
   retargeting) — no unrelated rewrites.
3. A genuinely new file with no predecessor belongs in a separate `Create` card, never folded
   into the `Rename` pair.
4. Never write the relocated file from scratch and delete the original — that loses
   git history exactly as an unstructured create+delete pair would.

## verify:

```bash
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
