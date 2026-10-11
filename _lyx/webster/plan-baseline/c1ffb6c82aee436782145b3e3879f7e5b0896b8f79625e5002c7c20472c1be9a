---
format: 5
approved: true
language: go
---

# Plan: orch-wake-policy

Batten takes over the mechanical part of the orch's landing routine and the halts whose way forward is mechanical, so the hub orch is woken only when its judgement is needed.
A new Land row between Run-Shed and Worktree-Teardown pulls the prime, runs the hub-wide `deploy` command, prunes the board and names the tasks the landing made ready; Run-Shed's `done` notice goes away, and the Land and Worktree-Teardown rows notify only on a failure or a newly ready task.
Batten retries the first Publish verify failure of a run on its own, fabric's bounded push retry widens to every plain push and to a remote `cannot lock ref`, and the orch role stencil, `CLAUDE.md` and the refusal spec move each step to its new owner.

## Card Index

1 — gitrepo-push-retry — gitrepo's plain push primitives retry a remote `(failed)` or `cannot lock ref` refusal and wrap an exhausted one in its own sentinel
2 — fabric-push-retry — Add's branch pushes and the archive tag push use gitrepo's retry, and the coalesced push Warns on an exhausted one
3 — publish-failure-target — the Publish failure record carries its target producer and write time
4 — goto-busy-kind — goto's held-run-lock refusal carries an envelope kind batten can read
5 — landing-deploy-key — landing.yaml gains the hub-wide `deploy` key
6 — drop-done-notice — Run-Shed no longer notices the orch of a done child
7 — row-failure-notice — Worktree-Teardown notices the orch once per failure episode, through a row notifier the Land row reuses
8 — land-producer — the Land producer: land lock, prime check, pull, deploy, prune and ready notice
9 — land-row — the Land row is registered, wired, guarded and placed between Run-Shed and Worktree-Teardown
10 — publish-retry — Run-Shed sends a child blocked on its first Publish verify failure to the record's target and resumes it
11 — orch-docs — the orch role stencil, CLAUDE.md and the refusal spec move each landing step to its new owner

## Shared Decisions

### Decision: land-row-pause

The Land row has no `on_stuck` (D1), so a producer cannot yield a pause through a self-route the way Run-Shed does.
A `lyx batten pause` seen during the Land row's land-lock or dirty-prime wait ends the wait with a Stuck that names the pause, halting batten blocked at Land with no notice.
Shed clears `pause_requested` only in a persist that records the run paused, at step 3 or in a goto, and a blocked persist leaves it set, so a bare `lyx batten run <slug>` after this halt would record the run paused and not call Land.
The Stuck therefore names `lyx batten goto <slug> --to Land`, which clears the flag and leaves the run paused at Land, then `lyx batten run <slug>`, which re-runs the row from the top; card 9 admits that goto and pins the sequence through shed.

### Decision: time-bounds

Values the decision record leaves to the plan: the deploy runs with a 30-minute timeout, the automatic Publish retry's goto and resume retries stop 30 minutes after the retry started, a row notice is tried at most `noticeMaxAttempts` times 30 seconds apart, the land-lock wait polls from one second doubling to 30 seconds, and the dirty-prime wait polls every 5 seconds within its ten-minute bound.

### Decision: run-markers-reset

Every marker that must last exactly one batten run (the row-notice, ready-snapshot and publish-retry markers) is named by a suffix in battenshed's `runMarkerSuffixes`, and `ClearRunMarkers` removes them all where batten seeds a fresh status, beside the terminal-opened marker's reset.

### Decision: land-steps-stuck

Every failed Land step, a seam read error included, is Stuck naming the step, never a hard error, since a resume re-runs the row from the top either way and the Stuck carries the reason into `lyx batten status` and the failure notice.

## verify:

```
go build ./...
go vet -tags integration ./...
go vet -tags tmux ./...
go vet -tags llm ./...
go test ./...
go test -tags integration ./...
```
