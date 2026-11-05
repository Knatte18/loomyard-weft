---
round: 1
focus:
  - "Fidelity, Decisions D2, D10, D11 and their `Added after Discussion` entries: settle whether cards 5, 6, 7 and 10 together carry the Beam-Await entry rule (re-check a standing ready record itself, otherwise lift the hold and start the dobby) and the fork refusal in `battencli`'s beam fork, and whether any card still has the dobby rewrite the land request."
  - "Card order, cards 5, 8, 9, 10, 11 and 15: settle whether each card's `Uses` and Intent lean on a symbol, seam or field (`Deps.RetiredRows`, `RetiredRowNames`, `WakeRecord`/`WriteWakeStamp`, `ReadLanding`, `DropLandingPullRequest`, `judgeAwaitingRun`, `beamPullRequestSide`, `ReadMergeState`) that a later card creates."
  - "Card 5 against card 12: card 5 removes the BeamDescribe producer and its wiring while `internal/loomcli/beamwiring.go` stays uncalled until card 12; settle whether the tree builds and the card's gate passes at card 5's boundary."
  - "Attack-surface question, cards 7, 12, 13, 14, 16 and 17 (new verbs, a fork refusal, a notice, and the `lym beam start` refusal relaxed for a held beam): settle whether each Intent states what the change can skip or let through and what bounds it."
  - "Reachability of prescribed refusals and halts, cards 8, 11, 13, 14 and 15: for each refusal or halt branch (retired-move step failure, closed or merged-at-another-head pull request, `RefusalPending/Other/Method` halts, `ready` refusals 3 to 6), settle whether the card names an input and a test that reaches it and whether an upstream check rejects that input first."
  - "Test economy, cards 9, 10 and 15: for each new test function (`TestEligibility_InterruptedPawlRows`, `TestWakeStamp`, `TestHubLandDeps_ReadyReads`) read the existing tests in `arbiterstop`, `shedrun` and `beamcli` and settle whether one already covers the behavior; name it if so."
  - "Edit-card shape, cards 5, 9, 10, 11 and 15: settle whether each `Edit` target that names symbols uses member glyphs, and whether each member whose parameters, results or receiver the Intent changes (`ResolvePullRequest`, `resolveWithParentRule`, `NewBeamAwait`, `markReady`, `landDeps` seams) carries the re-sign arrow."
  - "`Extends` and `Custom` typing, cards 9 and 8: settle whether `Extends` on `doc.go` files only adds, and whether any `Custom` card or `Prosa` group could be expressed with the other labels."
  - "PATTERN fit and PATTERN changes: settle whether the four PATTERN changes the overview declares are each carried by a card edit of `PATTERN.md` (`PATTERN-github-auth` in particular, and the overview's headroom), and whether cards 2, 3, 12, 13, 14 and 16 break `PATTERN-tracked-file-locks`, `PATTERN-remote-refs-lock`, `PATTERN-github-auth` or `PATTERN-durable-vs-ephemeral-state`."
  - "Board and filing, every card: settle whether any card edits this task's own board entry's body or scope, or files or promotes a task, for example in the `lym beam describe` refusal path that names `lym board upsert`."
  - "Verify lines: settle whether any card's `**Verify:**` line, or any test instruction in a card, names a raw test command rather than one `lym gate test` call."
---

Seed focus from the overview, the decision record and cards 5, 8, 9, 10, 11 and 15; the remaining cards were not read in this pass.
No entry restates a check the round's gate enforces, a `DependsOn`/`Produces` field, or an `ImpactSummary` on a `Rename`, `Move`, `Prosa` or `Custom` card.
