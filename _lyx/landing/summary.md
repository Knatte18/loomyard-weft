# tender: hub-wide pause, resume, revive and status

`lyx pause` and `lyx resume` become verbs of a new `tender` module, next to `revive` and `status`, so `lyx --help` lists modules only.
A pause no longer eats the deadline of a GitHub call it holds, and the long-lived processes that can restart safely pick up a replaced binary on their own.

Closes #606

## What changes

- `internal/pausecli` and `internal/pauseengine` are renamed `internal/tendercli` and `internal/tenderengine`; `internal/pauserecord` keeps its name.
  `lyx tender` has `pause [<slug>] [--grace <seconds>]`, `resume [<slug>]`, `revive [<slug>]` and `status`.
- `lyx pause` and `lyx resume` remain as shortcuts in `internal/shortcutcli/shortcuts.yaml`; the top-level root commands are gone.
  `pause` and `resume` behave as before.
- `lyx tender status` replaces `lyx pause --status`, which is now an unknown flag.
  It shows pause entries, network operations in flight and holds, every live agent session in the hub (pair, strand, state with cause and since time, whether a live pause covers it, the orch and other prime sessions included), and the stale-binary listing.
  It is read-only, so any caller may run it.
- `lyx tender revive` gives one fixed carry-on line to each session in state idle-stalled with cause `api-error`, for the hub or one pair.
  Today a network outage without a sleep needs `lyx shuttle send` per session by hand.
  A session covered by a live pause is reported as paused and left for `resume`; the orch gets a conditional `orchengine` notice instead of typed keys, dropped at delivery if the orch has recovered.
- `internal/shuttleengine` gains a conditioned send for `revive`.
  It refuses at once, sending no key, on a draft, operator activity, an unmatched turn start, an open rewind picker or a stray character, then waits at most one submit window for the turn start after landing.
- GitHub calls (#606): `pauserecord.Network` reports the held time, and `internal/githubclient` applies its 30-second client timeout itself in the transport, moving the caller's deadline later by the held time.
  A cancellation arriving before the caller's own deadline still ends the call.
  A GitHub deadline error stays transient, so the loom drive re-steps it once instead of halting `failed`.
- `lyx orch watch` re-execs itself through `selfexec` on a replaced binary, at a tick with no cycle step in flight and no live hub pause entry.
- `lyx reed attach` on Unix execs the tmux client instead of spawning it, so no lyx process lingers.
  It must be the last argv of any shortcut; the shortcut guard checks this.
- The stale-binary listing (Linux, through `internal/proc`) names hub processes whose executable is a deleted file with the running lyx's base name.
  Other platforms report it as unsupported.

## Design decisions to know

- `revive` runs each conditioned send inside `pauserecord.Update` with a no-op mutate, because shuttle's send does not consult the pause record.
  A pause therefore cannot slip in between the coverage read and the send; concurrent record writers wait at most one send's windows.
  A revive line whose turn start is never seen is reported as such and escapes the pause's interrupt record; `status` shows it busy under the live entry.
- Drain, a concurrency cap and a restart verb are out of scope.
  Mid-work processes (`lyx batten run`, `--until-stop` steps, agent sessions) are listed only.
- The watcher's lock does not survive the exec; the new image re-acquires it with the existing retry, and a racing `lyx orch start` ends with one watcher.
- Docs, `PATTERN-cli-cobra` background, test budget rows and the sandbox core suite are updated for the rename and the new verbs.

## To check by hand

- `held-page-row-page1-30ms-slack` (design): the held-page row in `internal/githubclient/githubclient_test.go` leaves 30 ms of slack on page 1; check it is not timing-flaky.
- `paste-row-unreachable-apierr-parser` (consistency): a paste row in the `revive` conditioned-send tests, `internal/shuttleengine/conditionedsend_test.go`, may be unreachable through the API-error parser; confirm the row exercises what it claims.
- Run `lyx tender revive` and `lyx tender status` against a live hub with a stalled session; the real network-outage path was not run.
- Run `lyx reed attach` on Unix and confirm no lyx process remains after tmux attaches.
- The verify gate first failed `TestTierPurity_UntaggedTestsSpawnNothing` on function-local sleep durations in `internal/githubclient` and `internal/selfreportengine`; they were moved to file-scope constants with the lengths unchanged.
