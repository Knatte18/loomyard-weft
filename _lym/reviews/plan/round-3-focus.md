---
round: 3
focus:
  - "Card 21's `hand_over` trigger: `RunView.PairPresent` is `markerExists(pair.WorktreePath())` in `internal/battencli/dobbyreads.go`, a path check, while `internal/pairteardown/handover.go` `HandOver` removes the pair only when the teardown reads it as a registered linked worktree. Settle whether a run the catch-up has handed over can still list as `hand_over` with `PairPresent` true and so owe the catch-up on every tick, and whether card 21 and card 20's `DobbyPairSlugs` read the same predicate for a pair standing here."
  - "Card 15 and `internal/tenderengine/resume.go` `Resume`: it judges the verdict, calls `req.Clear` (clears and pushes the run's pause intent), then runs `liftEntries`. Settle whether card 15 orders the refusal for a slug only a `bolt-hold` entry covers before `req.Clear`, so a refused `lym tender resume <slug>` leaves the pause intent standing and pushes nothing, and whether a planned test reaches that."
  - "Card 23's bring-up in `internal/orchcli/start.go` `pauseOwnedRuns` and `internal/orchcli/cli.go`: the card deletes the `pauseOwned` seam and ends each entry's processes through `pairteardown.EndSubjectProcesses(prime, slug, endDobby)`. Settle whether the card names the replacement seam, where the `endDobby` beam flag is read given that `fabricengine.EntryOutcome` carries no beam flag, and what the `owner_pauses` report key and the \"was not paused\" warning texts become."
---

Round 3 re-checks the three areas round 2 found open.
Every other area was cleared in round 2 and is out of focus unless the fixes touch it.
