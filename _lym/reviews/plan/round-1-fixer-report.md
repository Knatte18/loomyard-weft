# Fixer report — plan, round 1

Every finding's premise held against the plan and the code; all sixteen are fixed, two of them differently from the proposal.

## Changes

- **F1 (BLOCKING), fixed as proposed, then tightened by the self-check.**
  Card 4's `RelayState` lists `{PID, StartTime, FirstReadAt, FirstReadRefusal, LastSlackOK, StartFailures, NextStartAt, LastSpawnAt}`.
  Card 9 step 3 rewrites only `PID`, `StartTime`, `FirstReadAt` and `FirstReadRefusal`, keeping `LastSlackOK` and the starter's three fields, and every `RelayState` write is a locked read-modify-write through `internal/state`.
  `starter.ensure` records `LastSpawnAt` at every spawn; a failure counts when no accepted first read was written after `LastSpawnAt`, whether the relay died before or after step 3.
  The self-check's fix-proposal finding added: a start counts and spawns only once due at `NextStartAt`, every spawn sets `NextStartAt` at least one minute out, so a starting relay is never judged and each spawn counts at most one failure.
  `TestEnsureRelay` gains the step-3-then-die row ending in `gave-up`, the died-before-step-3 row, and a row for ticks inside a wait.
- **F2 (MEDIUM), fixed differently.**
  Not made: judging only a state whose `StartTime` is at or after the await's start. The relay `away` waits for starts before the await begins: `EnsureRelay` waits on the intermediate, which spawns the relay before it exits, and a `live` relay started earlier still. That rule would ignore the very relay it waits for.
  Made instead: `AwaitFirstRead` reads any gone relay's state as not yet written, whatever refusal it records, and drops the refuse-at-once branch; at the bound it names a live relay's recorded refusal, or the log.
  The `TestRelayWaits` row is as proposed: a stale gone relay's refusal is ignored and the await returns once a fresh `FirstReadAt` is written. Card 10's `away` names "a live relay's recorded creel refusal".
- **F3 (MEDIUM), fixed differently.**
  Not made: building the prime location from `--hub-path` through `fabricengine.PrimeName`. `PrimeName(l *lymcwd.Location)` runs `git worktree list` at `l.AnchorPath()` (`internal/fabricengine/worktreelist.go:150`), and a hub path names no worktree to run it from.
  Made instead: every relay argv carries `--prime <prime name>`, told by the caller of `EnsureRelay`/`EnsureRelayWhileAway`, whose signatures, and `SpawnRelayFromIntermediate`'s and the starter's, take `primeName`.
  - Card 10: `preRun` resolves location and prime name (`PrimeName`) for every verb but `relay`; for `relay` it resolves no location, as creel's `daemon --hub-path` does. The relay run builds the prime with `lymcwd.NamedWorktree(hub, name)` and builds `NewProgramClient`, `DobbyStart`, the pause geometry and `SlackGeometry` over it.
  - Callers: card 11 passes `location.WorktreeName`, card 12 `prime.WorktreeName`, card 13 the `primeName` `hubAnchorDeps` already resolves (`anchordeps.go:75`). The overview's argv decision names `--prime`.
  - Test: a `TestSlackCLI` row installs `preRun` over a resolver that refuses every cwd and reaches the relay run from a temp hub directory.
- **F4 (MEDIUM), fixed as proposed.**
  Card 10 states what `relay --fork` lets through and its bound, and that off Linux a by-hand run is accepted as `SLACK`, bounded by creel's environment-only identity. Card 9 step 4's bound says Linux.
- **F5 (MEDIUM), fixed as proposed.**
  Card 8's `read` records whether its handler ran; an `ErrWaitTimeout` after it ran is a lost confirmation that keeps the record and returns the floor. Card 3's `ReadWait` doc says its timeout spans the wait, the handler and the confirmation. `TestOutboundRead` gains the row.
- **F6 (MEDIUM), fixed as proposed.**
  `CheckBinary func(beforeExec func(), failed func(error))`; step 6's `failed` logs the exec error before it retakes the lock.
- **F7 (MEDIUM), fixed as proposed.**
  Card 3 moves `internal/creelengine/doc.go` to **Edit:**, rewrites its Refusal code section to name both coded refusals, and has `resolveSender`'s comment name both sentinels.
- **F8 (MEDIUM), fixed as proposed.**
  - Card 11 edits `orchcli/doc.go`, which gains the relay bring-up step, and `orchengine/doc.go`, which gains the presence line on the pointer.
  - Card 12 edits `beamcli/doc.go`, for `Deps.Tick` and its Bound, and `docs/overview.md`'s beamcli line.
  - Card 13 edits `fabriccli/doc.go` and `anchorCommand`'s `Long`, which gain the relay restart and `relay_start_error`.
- **F9 (LOW), fixed as proposed.**
  Step 6 retakes the lock with `lock.AcquireWriteLockWithin(geom.LockPath, lock.ProbeGrace)`. The failed-exec row of `TestRunRelay` covers a lock held only for a probe's grace.
- **F10 (LOW), fixed as proposed.**
  `TestInboundText` and `TestOutboundText` are dropped. Their rows move into `TestInboundPoll` and `TestOutboundRead`.
- **F11 (LOW), fixed as proposed.**
  Card 3 drops the failing-handler `ReadWait` subtest, which leaves two subtests. The remaining `ReadWait` subtest asserts the delivered message is acknowledged.
- **F12 (LOW), fixed as proposed.**
  Card 7's fake `Mailbox` scripts `ReadWait` answers and records acknowledgements from the start. Card 8 uses it unchanged.
- **F13 (LOW), fixed as proposed.**
  The login row of `TestSlackCLI` adds an empty standard input. It refuses, calls no `auth.test` and writes no token file.
- **F14 (LOW), fixed as proposed.**
  `TestSlackGeometry` adds a row that clears `HOME`, `XDG_CONFIG_HOME` and `LOCALAPPDATA` and asserts the error names them.
- **F15 (LOW), fixed as proposed.**
  Card 4 edits `pattern/PATTERN-cwd-resolution.md`, whose Bound packages list gains `slackengine`.
- **F16 (NIT), fixed as proposed.**
  Setup's `missing_scope` refusal names the bot scopes itself.

## Self-check

- fix-consistency, MEDIUM, `Hub.Paused` over `PauseGeometry(prime)` reads a beam orch's pause through the prime's slug: fixed as proposed. Card 10 reads the pause geometry of the receiving orch's worktree, using `NamedWorktree(hub, slug)` for a beam.
- fix-consistency, LOW, the `preRun` splice left a lowercase sentence that contradicts the new row: fixed as proposed.
- fix-consistency, LOW, the overview's argv decision is missing cards 11 to 13 under Applies to: fixed as proposed.
- fix-consistency, LOW, the ImpactSummaries of cards 3, 11, 12 and 13 omit the doc edits: fixed as proposed.
- fix-consistency, LOW, card 8's lost confirmation with no post has no return value; merged with fix-proposal's F5 finding: fixed as proposed. The floor wait applies whether or not a post succeeded, a row covers a handler that posts nothing, and the sibling activity bullet now says only that a successful post returns the floor.
- fix-claims, LOW, "a gone relay's recorded refusal is an earlier relay's" overstates the case: fixed as proposed. The rationale is narrowed to "never the strand-pane refusal of a relay this start launched".
- fix-proposal, MEDIUM, the F1 count fires on every tick, including while a relay is still starting: fixed as proposed. The count and the spawn run only once due, every spawn sets a `NextStartAt` at least one minute out, each spawn counts at most one failure, and a row covers ticks inside a wait.
- fix-proposal, LOW, F5's lost confirmation with no post: merged above.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
