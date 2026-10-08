# Webster round 2 — fixer report

All four findings validated against the code and fixed, one commit each.
`go build ./...`, `go vet ./...`, `go test ./...` (untagged) and `go test -tags integration` over `shuttleengine`, `websterengine`, `webstercli` and `loomcli` pass; `lyx loom lint-comments` is clean.

## Fixed

- **F1** (`8740f19ed`) — `recoverSpawn` now stops the prior strand only when `prior.StrandGUID != ""`, as `BeginBatch` does, and `reclaimEntryTimeStrands` skips a non-terminal recovery record with no guid the same way.
  `awaitTerminal`'s cleanup of its own finished strand calls `Stopper.StopStrand` directly, without the respawn Warn.
  The respawn Warn stays on the entry reclaim, begin-batch, recover-batch's prior strand and reset.
  Regression row in `TestRecoverSpawnOrAttach`: a failed fork batch, which records no strand, spawns without stopping one.
- **F2** (`be3a19817`) — `sendVerified` counts the typings it played, and the never-appeared message reports that count.
  When the submit window, cut to the send's deadline, has closed, the message says the window closed before the text appeared instead of blaming a swallowed input.
  New row in `TestRun_Send_ConfirmsSubmission` pins the window-ended path, which no row reached: a 1 s window ends the loop after one typing.
- **F3** (`72ae0880c`) — `ErrSubmissionNotLanded`'s doc now covers a send whose window closed before typing began.
  The `handleGatedBoundary` re-prompt comment now says the send cleared its own text where it could, and that a box it left occupied fails the re-send busy for an engine with the idle reading, until `maxUnsentReprompts` ends the loop.
- **F4** (`7649d466d`) — `send_ready_timeout_s` and `submit_confirm_timeout_s`, in `template.yaml` and on their `Config` fields, now name the engine capability they bind (idle reading, input-box reading) and that a gate send's wait ends at the run's deadline.
  The template text names no provider, per `PATTERN-shuttle-provider-seam`.

## Disputed

None.

## Deferred

None.
