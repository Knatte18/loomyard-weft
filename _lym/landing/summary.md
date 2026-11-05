# Beam orch drives landing inside Beam-Await; ready is final

A beam's landing work moves into Beam-Await, driven by the beam's orch with the operator.
Pawl drops Parent-Align, Beam-Describe and PR-Review and runs Preflight, Record-Write, Beam-Up, Beam-Await, Finalize, Beam-Down.
Those rows suited tapestry and darn, which are autonomous; a beam is iterated with the operator, and a fixed row after Beam-Await forced a cancel or a reopen whenever the beam needed more work.
Tapestry, darn and sampler keep Parent-Align, PR-Review and their Finalize pull-request handling unchanged.

Closes #57
Closes #56
Closes #47

## What changes

- **New verbs for the orch.**
  `lym beam describe <slug>` writes `_lym/landing/summary.md` whole (each beam commit with its pull request number or landing commit, merge time and summary) and stamps it.
  `lym beam pr open|update|status|close` opens, edits, reads and closes the beam's pull request over `internal/pullrequest`; `status` reports `mergeable_now` and the causes when it is false.
  The orch never merges a pull request and there is no `lym beam pr merge`.
- **`lym beam ready` is the point of no return.**
  `lym beam reopen`, the reopen request and PR-Review's reroute are removed.
  Ready now refuses, each with a way forward, a run not at Beam-Await, open work under the beam, parent commits not merged in, a missing or invalid description, a required pull request not recorded, and a recorded pull request that is stale, closed, or not mergeable now.
- **Hold boundary moves to Finalize.**
  `boardengine.RunHolds` holds the beam from Finalize on.
  Between ready and the accept, the dobby starts no member, and the member run fork in `internal/battencli/beamfork.go` refuses one, naming `lym batten cancel <member> --remove`.
- **Beam landing record.**
  A tracked record in the beam worktree's landing directory carries the description stamp and, while a pull request is recorded, its number, URL, pushed head and announce head.
  It names its beam; a record naming another slug reads as none and is never removed.
- **Member landings while the pull request is open.**
  The dobby posts one notice to the beam's orch per landing into the beam, with no `notify_on_land`, naming `lym beam describe` then `lym beam pr update`.
  The last announced head is never-tracked state under `.lym/beam/`.
- **Finalize lands a pawl run by the record.**
  With a pull request recorded, it merges in the parent, runs the gate, pushes, squash-merges that pull request on GitHub without editing its title or body, pulls the parent, completes the parent-side landing, then drops the pull request from the record.
  Without one it uses an open scratch-record pull request, or finds or opens one where a pull request is required, or squash-merges locally for `on` or `off`.
  A pending refusal is retried within `prMergeRetryWindow`; any other refusal halts naming GitHub's message, `lym beam pr status <slug>` and `lym loom resume`.
  The entry's `pull_request` field is not read for a beam (#56).
- **Beam-Await survives a loop exit (#57).**
  Beam-Await, Beam-Up and Beam-Down get the re-invoke interrupt policy.
  The driver restarts its repair count only when a row's completed-wake stamp is at least ten minutes past the last re-step, so a long wait survives every exit while a crash loop still reaches the tackler.
  Beam-Await's entry finding the hold named Finalize with no land request re-checks a standing ready record itself, or lifts the hold and restarts the dobby.
- **`lym beam start` in landing (#47).**
  It brings back the orch of a beam held by its own pawl run, Finalize and Beam-Down included, and leaves the dobby down.
- **In-flight pawl runs.**
  A recipe may declare retired rows; pawl maps Parent-Align, Beam-Describe and PR-Review to Beam-Await.
  Shed moves such a run once, pauses it, lifts the hold, ends a live PR-Review session, carries an open scratch pull request into the landing record, removes the ready record, land request, dobby-stop marker and reopen request, commits the carry, and notifies the beam's orch.
  A run at Finalize or later is untouched.
- The orch role text and the `ready`, `pr` and `describe` help carry the landing workflow and the finality of ready.
- `PATTERN.md` entries for pull-request mode, GitHub auth, heddle instructions and owner machine are updated.

## Design decisions

- #47's point that a push lock whose holder died is taken as stale is dropped: a flock is freed when its holder dies, so the `fabric.push.lock still held after 30s` seen was a live push left running across the restart.
  How long a push holds its locks belongs to the separate task `push-holds-remote-refs`.
- The board cannot read the tracked ready record, so the dobby and the fork precondition enforce the ready hold until the entry status takes over.
- Recovery after a crash between the accept and Done lives in Beam-Await's entry, not in the dobby, because the dobby's persisted land episode stops it from writing the land request again.
- The retired-row map is a recipe-level key read by the recipe parser; the pawl side effects hang on a seam the pawl wiring tells, so shed stays recipe-agnostic.

## To check by hand

- Beam-Await's entry may miss the re-check for a halted run found with the hold named Finalize (`internal/pawlengine/beamawait.go`).
- `lym beam ready`'s refusal may quote `lym loom resume` as a way forward where resume does not apply (`internal/beamcli/land.go`).
- A merged pull request whose head differs from the beam's: confirm the "lacks no beam commit" rule in `lym beam ready` and Finalize's landing of it agree (`internal/beamcli/land.go`, `internal/landingshed/finalize_pr.go`).
- A parent with an upstream branch holding commits the beam lacks has no test for ready's refusal 3 (`internal/beamcli/land.go`).
- A scratch-record pull request override in Finalize may merge a pull request that is closed (`internal/landingshed/finalize_pr.go`).
- Not run by hand: a live `lym beam pr open|update|status|close` against a GitHub repository, a pawl run moved off a retired row by a new binary, and a restart during Finalize followed by `lym beam start`.
