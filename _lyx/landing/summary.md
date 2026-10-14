# hibernate: lyx hibernate trigger, plus lyx p and lyx r

The operator can now queue a machine hibernate from the phone through the orch, and lyx's Go code carries it out once every session in the hub is safely paused.
No LLM session runs `systemctl hibernate`, so none is left hanging mid-call when the machine goes down.
`lyx p` and `lyx r` join `lyx pause` and `lyx resume` as one-letter shortcuts.

## What changes

- New `lyx hibernate` module (`internal/hibernatecli`, `internal/hibernateengine`) with `request`, `cancel` and `status`, and a hidden `watch` verb that `request` spawns as a detached daemon.
- `request` takes no trigger (fire as soon as the pause is confirmed), `--after <slug>` (fire once that slug's batten run has finished its Land row) or `--after-all` (fire once no batten run in the hub reads running).
  The two flags are exclusive.
- One request per hub lives in an ephemeral file under the hub's `.lyx`, written under its lock, so two racing requests yield one daemon.
- On firing, the daemon runs tender's hub-scope pause, waits until the hub reads confirmed-paused (every session idle or dead, orch included, no network operation or land lock in flight), runs `systemctl hibernate`, detects the wake through `pauserecord.SleepWatch`, lifts the pause entry it owns and queues one orch notice.
  A pause that never confirms (120 re-reads, five seconds apart), a non-zero exit or no wake within five minutes ends the request `failed`, lifts the owned entry and notifies the orch.
- `tenderengine` gains a lift of one pause entry by scope, caller and paused-at time, and its pause reports the entry it added.
  `tendercli` exports its hub wiring as `HubDeps` for the new module.
- Spawned Claude sessions are denied `systemctl` with `sleep`, `hibernate`, `suspend`, `hybrid-sleep` and `suspend-then-hibernate` in `claudeengine/settings.go`; the steer points at `lyx hibernate request`.
- The orch role stencil routes an operator's hibernate request through `lyx hibernate request`.
- `internal/shortcutcli/shortcuts.yaml` gains `p: [tender, pause]` and `r: [tender, resume]`.
- Docs: `doc.go` for both new packages, the module table in `docs/overview.md`, the `tenderengine` and `shortcutcli` docs, the refusal spec and the `PATTERN-no-denied-recovery` and `PATTERN-daemon-wakeups` pattern files.

## Design decisions

- The daemon owns at most one pause entry, by identity.
  When a hub pause is already live, the request owns none and neither the daemon nor `cancel` lifts that pause.
- Right before the hibernate call the daemon re-checks that the request is not cancelled and a live hub entry still stands; an operator `lyx resume` in that window ends the request `cancelled` with no hibernate.
- `cancel` refuses while a live daemon is in `hibernating`; a dead daemon's request cancels in any phase, which is the way out after a failed resume from hibernation.
- `--after-all` counts halted and awaiting runs as out of flight; an `--after <slug>` whose run halts before Land stays `waiting` and `status` says why.
- The daemon sends no key or text itself; every interrupt is the pause engine's, under its existing guards.
- An agent of a pair is refused on `request`, `cancel` and `watch`; the orch and an external caller may run them.
- Linux only: elsewhere `request` and `cancel` refuse as unsupported and `status` still reads.
- The `hibernate` module is excluded from the sandbox suite, since its daemon hibernates the host.
- The pause grace, the confirmation window and the wake window are fixed in code with no config key.
- The sleep deny is a guardrail, not a barrier: `bash -c`, `eval`, `systemctl start` of a sleep target and routes such as `busctl` or `rtcwake` pass.
  It is Claude-engine settings only; another engine needs its own.
- The `p` and `r` names are the board entry's proposed names, picked without the operator's confirmation, which the board entry reserves.

## To check by hand

- The `p` and `r` shortcut names: confirm or rename.
- Run a real `lyx hibernate request` on the target machine; no test runs `systemctl`, and `systemctl hibernate` may need polkit permission.
- The run's plan-level verify was not run: run `lyx gate test ./...` and `lyx gate test --tags integration ./...`.
- Files the build touched outside the plan's declared targets: the `internal/hibernateengine` request, verbs, watch and test files, `internal/tendercli/cli.go`, `internal/hibernatecli/cli.go`, `cli_integration_test.go` and `testmain_test.go`, `cmd/lyx/daemonwakeups_test.go`, `pattern/PATTERN-daemon-wakeups.md` and `cmd/lyx/testdata/test-budget.yaml`.
- The `request` command's `Use` string omits its flags, because `TestCLIForm_EveryCommand` rejects the long form.

## Known risks

- A second hub on the same machine is not paused before the hibernate, so its sessions would be cut mid-call.
- The operator's own typing into a session between confirmation and the hibernate call is outside lyx's control.
- `--after <slug>` fires after Land's deploy, so the daemon runs from a replaced binary and `lyx tender status` lists it among the processes running one.
- A failed resume from hibernation boots cold with the daemon gone and its pause entry live; the hub stays paused until `lyx hibernate cancel`.
