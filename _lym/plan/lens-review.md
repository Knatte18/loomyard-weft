# Lens review

Every lens returned a report; each is saved beside this file as `lens-review.lens-<lens>.md`.
Duplicates are merged into the first line that carries them, which names the others.

- coverage 1 — BLOCKING — fixed as proposed: the overview's `## PATTERN changes` drops the withdrawn `PATTERN-reap-children` and `PATTERN-slack-boundary` bullets the parent removed from the record (creel #766), and the first Shared Decision's rationale states the record as it now stands.
- coverage 2 — MEDIUM — fixed differently: the proposal asks for a row that drives the double fork end to end, which needs a real pane and a creel daemon placing the relay (a tier2 rig) for what the parts already pin; instead card 9's `TestEnsureRelay` gains a `spawnRelay` row (the `--after <pid>` spawn, not waited on), card 10's `TestSlackCLI` gains `relay --fork` and `relay --after` rows, and `TestRelayWaits` keeps `AwaitReparent`'s rows, while creel resolving a process in no pane as the claimed program is creel's existing behavior (`internal/creelengine#resolveSender`).
- coverage 3 — MEDIUM — fixed as proposed: card 12 gains `TestHubDobbyDeps_PrimeTickEnsuresRelay` (tier1), whose prime tick with presence `slack` reaches the spawn (`ErrSpawnUnderTest`) and with `terminal` answers false and nil, so a tick that never calls `EnsureRelayWhileAway` fails it.
- coverage 4 — MEDIUM — fixed as proposed, merged with forced-edits 2 and guard-reach 4: card 11 wraps `ChooseStartPrompt`'s prompt inside `orchCLI.launchOrch` too, and gains rows in `TestWatcher_HandoffCompleteClearsThenResumes`, `TestCompact_RequestedCycleClearsRequestAndCompactsAfterNote` and `TestStart_ResumesTheRecordedSession` for the clear, compaction and launch sites.
- coverage 5 — MEDIUM — fixed as proposed, merged with row-validity 1: card 9's `RelayDeps` gains `PID` and `ReexecLockWait`, and `TestRunRelay` gains a re-exec row against a fresh-start row with distinct fake pids.
- coverage 6 — MEDIUM — fixed as proposed: card 10's `TestSlackCLI` gains a `status` row printing presence, liveness, pause coverage, channel, operator and `LastSlackOK`.
- coverage 7 — LOW — fixed as proposed, merged with row-validity 6: card 13 calls `StartRelay` once `EndDobbies` was called, an error included, and `TestChangeAnchor` gains an `endErr` row logging `start-relay` and a `lockErr` row logging none.
- coverage 8 — LOW — fixed as proposed: card 9 and card 10 state the relay verb exits 1 with `AwaitReparent`'s error, taking no lock, and `TestSlackCLI` gains that row.
- coverage 9 — LOW — fixed as proposed: card 7 gains an escaped-answer row and a refused-answer row, and card 8's row names a `5xx` beside the transport failure.
- coverage 10 — LOW — fixed as proposed: card 9's accepted-first-read row asserts `Register` is called once.
- coverage 11 — LOW — fixed differently, merged with row-validity 5: the proposal's row through `wireCLI` cannot be driven by the start harness, which never runs `wireCLI` (`internal/orchcli/startdobbies_test.go:60`); card 11 moves the prime-only choice into `relayStarter(location, beam)`, which `wireCLI` calls, and `TestStart_StartsTheDobbiesFirst` gains its two rows.
- coverage 12 — LOW — fixed as proposed: card 7's poll keeps the presence its previous poll read and counts any change as activity, with a row for a change made by another writer.
- row-validity 2 — MEDIUM — fixed as proposed: card 9 declares `starter.ensure` and `starter.spawnRelay` over a `starter` with a fake spawn and clock, which `TestEnsureRelay` builds, so no package-level seam stops `t.Parallel`.
- row-validity 3 — MEDIUM — fixed as proposed: card 10 declares `slackCLI.command`, which `Command`'s pre-run wires and the tier0 test drives over a hand-built `slackCLI`, and states that no seam is nil once wired.
- row-validity 4 — MEDIUM — fixed as proposed: card 11's `ResumeContext` returns a pending pointer as stored, without a second `WithPresence`, and a row pins the line once.
- row-validity 7 — LOW — fixed as proposed: card 3 adds the `errors.Is` assertion to the existing `Linux_ProgramClaimFromANonOrchPaneIsRefused` row instead of a new row.
- row-validity 8 — LOW — fixed as proposed: card 10 drops `TestPaneRefusal`; its unreadable-strands and off-Linux `StrandName` rows join `TestSlackCLI`.
- row-validity 9 — LOW — fixed as proposed: card 10's `away` row asserts only that the presence change follows the await, and card 8's `TestOutboundText` drops the not-posted rows `TestOutboundRead` holds.
- row-validity 10 — LOW — fixed as proposed: `TestRunRelay` gains a row that a successful Slack call writes `LastSlackOK`.
- forced-edits 1 — BLOCKING — fixed as proposed: card 10, which mounts the module that `TestSandboxCoverage_AllModulesCoveredOrExcluded` reads from `newRoot`, adds a `slack` entry with its reason to `cmd/lym#excludedModules`; the config module is not a root command, so card 5 needs none.
- forced-edits 3 — LOW — fixed as proposed: card 11 states that `orchPaths` is the constructor every orch verb and the watcher use (`PrimePaths` delegates to it) and that `orchroute.Paths`'s other callers render no pointer.
- forced-edits 4 — LOW — fixed as proposed: card 4 lists `internal/hubgeom/doc.go` under `Extends` with the teller sentence.
- forced-edits 5 — LOW — fixed as proposed: card 2 edits `internal/lymdirs/dirs.go`'s header to name the Slack directory.
- forced-edits 6 — LOW — fixed as proposed: card 10 adds `slackcli.RunCLI` and `RunCLIIn` to `cmd/lym/seamsignature_test.go`'s two lists, a file with no symbol.
- forced-edits 7 — LOW — fixed as proposed, merged with guard-reach 6: card 3's end-to-end subtest is generated on Linux only.
- way-forward 1 — MEDIUM — fixed as proposed: card 9's `CheckBinary` takes `selfexec`'s `failed` callback, which retakes the lock or exits when another relay took it, with rows.
- way-forward 2 — MEDIUM — fixed as proposed: card 9 caps the `whileAway` restarts with a doubling wait from one minute to 30 and a stop after five relays in a row that never made their first read (`gave-up`), reset by an accepted read or an explicit start; card 12 logs `gave-up` at Warn naming the log and `lym slack start`.
- way-forward 3 — MEDIUM — fixed as proposed: card 7 posts one `[slack relay]` notice per episode to the prime orch for an inbound `*APIError`, naming `lym slack login` or `lym slack setup`, with rows.
- way-forward 4 — LOW — fixed as proposed, merged with guard-reach 3: card 10's `setup` maps `channel_not_found` to the invite refusal and names ways forward for `missing_scope` and `user_not_found`, with rows.
- way-forward 5 — LOW — fixed as proposed: card 10's `login` refusal names the OAuth page and `lym slack login`.
- way-forward 6 — LOW — fixed as proposed: card 9's `AwaitFirstRead` refusals name `lym slack away`, which `away` passes on.
- way-forward 7 — LOW — fixed as proposed: card 9's `ErrRelayInStrandPane` names `lym slack start` from the operator's terminal or an orch's pane.
- way-forward 8 — LOW — fixed as proposed: card 9's `AwaitFirstRead` refuses at once when a gone relay recorded a refusal, with a row.
- way-forward 9 — LOW — fixed as proposed: card 10's `stop` names a re-run of `lym slack stop`, with a row.
- way-forward 10 — LOW — fixed as proposed: card 5's refusal names `lym config slack`.
- guard-reach 1 — MEDIUM — fixed as proposed: card 9's first read hands mail back with the sentinel `errFirstReadAccepted`, read as accepted, with a row for a first read holding unread SLACK mail.
- guard-reach 2 — MEDIUM — fixed as proposed: card 8 treats `ErrWaitTimeout` as an empty read, with a row.
- guard-reach 5 — MEDIUM — fixed as proposed: card 9's fresh start waits `lock.ProbeGrace`, with a row for a lock held only for a probe.
