# Fixer report — plan, round 1

## Changes

- F1 (card 2) — fixed as proposed: every descendant signal follows a fresh start-time comparison; the grace poll counts a pid with a differing start time as gone; the SIGKILL phase repeats every start-time and group check; `EndProcess` re-compares before its SIGKILL. `TestEnd` gains rows for a descendant reused at the SIGTERM and between SIGTERM and SIGKILL, and an `endProcess` row for a pid reused before the SIGKILL.
- F2 (card 11) — fixed as proposed: the gone-worktree path takes `localReading` (`removeVerdict` over an empty report with `Unreachable` set), so `owned_elsewhere` and `ended` remove locally with the dropped `--remote` named. `TestRemover` gains the gone-worktree row with a local foreign owner and keeps the row with neither reading.
- F3 (card 17) — fixed as proposed: a gated waiting turn end with only unawaited shells returns `OutcomeDone` only when every output file exists, and otherwise the held turn end of that waiting turn end. A new `TestPollEventsTick_OutstandingBackgroundWork` row covers the gated run with an output missing. The `//testtiming:keep` reason wording follows.
- F4 (card 10) — fixed differently: the review proposed passing a marker of another registered pid when the holder is unknown and no parent lock record names the slug. The self-check showed that branch can never decide an outcome, because a marker exists only under a pause and the pause case already passes. It also showed an unknown holder is usually the registered Go driver (`internal/pairteardown/loop.go:118`). So the holder is now read as the unfinished step's pid or else the slug's registered driver (`driverPID`), and only a marker naming the holder's own pid passes. The rows are rewritten to match.
- F5 (card 10) — fixed as proposed, then widened by the self-check: a paused holder refused over a parent lock record, known or not, names `lyx tender resume <slug>` (or `lyx tender resume` for a hub entry). Under a pause held for another machine it names retrying once the holder ends, and never lifting that pause. The wording lives in the new `runLockRefusal`, and the refusal rows assert it.
- F6 (card 10) — fixed as proposed, then widened by the self-check: an unknown holder, now only an unregistered process, names the run lock path and, by `goos`, `lyx tender status`'s replaced-binary part or `ps` on Linux and Task Manager on Windows. `runLockRefusal` takes `goos`, and the refusal rows assert both.
- F7 (card 17) — fixed as proposed: `internal/websterengine/doc.go` joins the Edit list, and the Docs paragraph rewords its `# Background shells in Master's wait` section to D11. The self-check added `wait.go`'s file header and `expiredTurnEnd`'s doc comment to the rewording. `validate-plan` is ok.
- F8 (card 2) — fixed as proposed: `Register` locks at the record path plus `.lock`. `List` reads only `.json` names and removes a pruned record's lock file with it. `TestRegisterAndList` asserts the lock file is gone.
- F9 (card 11) — fixed differently: the review proposed naming `lyx tender pause <slug>`. That pause does not unblock the removal, because a live branch in a run-scope report is always the parent pair's (`internal/fabricengine/originreconcilemoves.go:379`, `target.slug != m.scope.Run`). The refusal now names the parent pair and the re-run once that pair is no longer live, or after `lyx tender pause` with no slug, under whose hub entry a live pair reads not live (`internal/pairteardown/doc.go:67`). A `TestRemoveVerdict` row covers it.
- F10 (card 13) — fixed differently: the review proposed putting the way forward (`lyx fabric remove <slug>` or `lyx reed down` while the pair exists, `lyx tender reap` once it is gone) into the Warn lines. Instead `ownedElsewhereEnder.run` wraps the end error and the `*runproc.SurvivorsError` with it, so every Warn line that logs the error names it and `TestOwnedElsewhereEnder` pins it. No shared log-capture helper exists outside `internal/logger`'s own tests.
- F11 (card 12) — fixed as proposed: `downCmd`'s `Long` text gains the pair readings, the refusal naming `lyx tender pause <slug>`, and `ended_processes`.

`lyx loom validate-plan` reads ok after every change.

## Self-check

- fix-consistency, MEDIUM, card 10 unknown-holder wording has no Windows branch: fixed as proposed (`goos` on `runLockRefusal`, Task Manager wording, row).
- fix-consistency, LOW, card 10 known-holder row missing its unpaused qualifier: fixed as proposed (rows rewritten with "unpaused and paused with a parent lock record").
- fix-consistency, LOW, card 13 Warn wording pinned by no test: fixed with own change (the way forward moved into the wrapped error, which `TestOwnedElsewhereEnder` asserts).
- fix-claims, LOW, card 2 absolute "no signal reaches a reused pid" claim: fixed as proposed (reworded to the last reading before each signal, naming the remaining window).
- fix-affected-set, LOW, `wait.go` header and `expiredTurnEnd` doc keep the old rule: fixed as proposed (card 17 Docs).
- fix-proposal, MEDIUM, F9's `lyx tender pause <slug>` cannot unblock a live parent branch: fixed with own change (see F9).
- fix-proposal, MEDIUM, F6's unknown holder is the registered Go driver: fixed with own change (the holder is resolved through `driverPID`, and the resume wording covers an unknown holder too).
- fix-proposal, MEDIUM, F5's resume wording wrong under an owner pause: fixed with own change (the refusal names retrying once the holder ends under a pause held for another machine).
- fix-proposal, LOW, F4's other-pid marker branch never decides: fixed with own change (branch dropped, see F4).

## Disputed

None.

## Deferred

None.
