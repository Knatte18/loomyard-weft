---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
  - id: F5
    disposition: fixed
  - id: F6
    disposition: fixed-differently
    reason: The proposal put every worktree whose origin does not read as a beam into tasks only when the origin reads, which would drop an unwired pair; a pair with no _lym junction stays a task pair, as before the diff, so its reed session is still downed.
  - id: F7
    disposition: fixed
  - id: F8
    disposition: fixed
  - id: F9
    disposition: fixed
---

## Changes

- F1: `detachedStopCommand` in `internal/beamcli/hubstop.go` builds the detached command and sets `Dir` to `prime.AnchorPath()`; the spawn logs its args and dir.
- F2: the `BeamDown` doc in `internal/orchcli/stop.go` and `internal/orchcli/doc.go` state that the strand's removal ends the orch's pane and everything below it, so the later steps run only for a caller outside it.
- F3: a `stop b --all` row with the exact refusal in `start_integration_test.go`; `TestDetachedStopCommand_RunsDetachedAtThePrimesAnchor` asserts the arguments and directory; `TestHubStopWorktrees_SplitsBeamsFromTaskPairs` (`hubstop_integration_test.go`) asserts the beam and the task pair land in their lists.
- F4: `TestRunHubStop` rows for a failed pause, worktree list, pair down, beam down, reap and lift, and for `ConfirmationLifted`, asserting call order, exact failure strings, report sizes, and that t1 is absent from `Pairs` after its down fails.
- F5: `beamDown` runs `endLiveDobby`, then always `endOrch`, and returns `errors.Join` of both errors; a row where `endDobby` fails asserts `end-orch` still runs.
- F6: `hubStopWorktrees` reads each non-main pair's junction and origin through `fabricengine`; a pair whose junction or origin cannot be read is left out of both lists and named in the returned error, which `runHubStop` records in `Failures`.
  Differently from the proposal: a pair with no `_lym` junction stays a task pair.
  The unreadable-pair branch has no test.
- F7: rows where `endOrch` fails in `TestBeamDown_StopsOrchThenTheDobbyThenTheOrchProcess` assert the error names the step and the worktree and the report keeps `OrchRemoved` and `DobbyEnded`.
- F8: `runHubStop` lifts only when `paused.Owned` is set and the outcome is not `ConfirmationLifted`.
- F9: the `internal/hibernatecli` tree line in `docs/overview.md` names the hub pause and confirmation.
- Also, from the self-check: `internal/beamcli/doc.go` states the anchor start, the unreadable-pair failure and the dobby-step failure.
- Gate repair: the `stop --all` sentence in `internal/beamcli/doc.go` that reached 120 characters is split into two sentences, one per line.

Commits are grouped by package rather than one per finding, since F1, F3, F4, F6 and F8 touch the same files.

## Self-check

- fix-code-correctness, LOW: an unwired pair was dropped from both lists; fixed differently, as F6's disposition says.
- fix-consistency, LOW: `internal/beamcli/doc.go` stale on the anchor start, the task classification and the dobby failure; fixed.
- fix-consistency, LOW: `docs/overview.md` line 850 lists only `request`, `cancel` and `status`; not fixed, it is a feature summary that does not contradict the tree line (disputed, premise false: it names no package contents).
- fix-proposal, LOW: the same unwired-pair drop; fixed with the first.
- fix-affected-set: no findings.

## Disputed

- Self-check finding on `docs/overview.md` line 850: `premise false`. The bullet describes the `lym hibernate` commands, which the hub pause did not change (`docs/overview.md:850`).

## Deferred

Nothing deferred.

## Rulings

No ruling reached this round.
