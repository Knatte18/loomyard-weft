# lym beam stop --all takes the hub down on this machine, behind a button

Today an operator stops a machine's share of a hub by hand: `lym beam stop` per worktree, `lym reed down` per pair and `lym tender reap`.
`lym beam stop --all` now does this in one command, and reed's prime button group gets a `stop` entry that runs it behind a confirm.
It ends every orch, dobby, run process and reed session of the hub on this machine, whoever owns each worktree, and writes no tracked record.

## What changes

- **`internal/hibernateengine`** — hibernate's pause-and-confirm becomes one exported function, `PauseAndConfirm`.
  It pauses the hub through tender with the ten-second grace, then reads the hub until the pause is confirmed:
  every session idle, no network call in flight and the land lock free.
  It owns only an entry it added, never lifts one, and returns the owned entry, the outcome and the last reason a reading withheld the confirmation.
  Hibernate's `carry` calls it through hooks (`Added`, `Confirming`, `Carried`) for its request file, so the hibernate sequence's behaviour is unchanged.
- **`internal/hibernatecli`** — `PauseHub` runs `PauseAndConfirm` over the hub's Deps with no request file, under a caller label the caller names.
- **`internal/orchcli`** — `BeamDown` ends the orch session's registered provider process as its last step, after the strand and the dobby.
  The report gains `orch_process_ended`.
  Every caller now ends that process: `lym beam stop <slug>`, a beam's cancel and pawl's beam-down row.
- **`internal/reedcli`** — the `down` verb's body moves into one method.
  The new `DownWorktree` runs that method for a worktree the caller names, so one process can take down the sessions of several worktrees.
  The verb's envelope is unchanged.
- **`internal/beamcli`** — `lym beam stop --all`.
  The caller's process re-starts the verb detached in the prime, with a hidden `--detached` flag, through `selfexec` with its spawn logged, and prints the detached pid.
  So the end of the session it was run from, the prime's, which goes last, does not cut it.
  The detached process runs the sequence and goes past every failed step, naming each in its report:
  1. a hub pause entry under caller `beam-stop` and hibernate's confirmation, through `hibernatecli.PauseHub`;
     a hub entry already standing covers the stop and stays, and an unconfirmed pause names what was still busy, then the stop goes on;
  2. each task pair's processes and reed session through `reedcli.DownWorktree`, which accepts a pair the hub pause covers;
  3. `orchcli.BeamDown` for the prime and each local beam;
  4. each beam's reed session;
  5. the run processes no run owns, through `tenderengine.Reap`;
  6. its own pause entry, if it added one, lifted by identity through `tenderengine.Lift`;
  7. the prime's reed session, last.

  A slug beside `--all` is refused, naming both ways forward.
- **`internal/reedengine`** — the default `prime` button group in both templates gains `stop`, which runs `lym beam stop --all`.
  Its confirm text reads "Stop every orch, dobby, run and reed session of the hub on this machine?".
  A hub that sets `buttons.prime` in its own layer keeps its own group and adds the entry by hand.

The package docs of the changed packages and `docs/overview.md`'s beam module line describe the new behaviour.

## Design decisions

- **The orch process ends through `runproc.EndRole`, not `runproc.EndSubject`.**
  The record names `EndSubject`, but over the prime's `HubOwner` record that would also end every other hub-owned process it does not spare, such as a gate or capture child.
  `EndRole` with `RoleOrch` ends exactly the registered Claude process and the processes below it, and it is the end `BeamUp` already uses for the same record.
  Like every runproc end, it spares the caller and its ancestors, so a `lym beam stop` run inside that orch's own session has finished every other step when it ends.
- **No separate coverage-map upkeep step.**
  The dobby builds no coverage maps today; `lym gate testmap` builds them on command.
  Ending the prime's dobby therefore leaves no upkeep process.
- **The detached process's report goes to lym's log.**
  The detached process has no terminal, so it logs its report, the unconfirmed reason and each failed step.
  The caller's own envelope carries only the detached pid.
- **The confirmation reads the hub as hibernate's does.**
  The requester is the caller's strand from the environment, and the orch's session is left out only when that strand is an orch's.
  A busy orch can therefore hold the stop for up to hibernate's ten-minute wait before the stop goes on.
- **The stop's own process is not registered in `internal/runproc`.**
  It works in the prime, a registered worktree, so `tenderengine.Reap` never reads it as an orphan, and no run's end reaches it.

## Bounds

The stop ends agent steps mid-step; every transition is already pushed or owed, so nothing pushed is lost and the cut step runs again.
After a stop, `lym beam start --all` or `lym s` brings the hub back.
A run whose loop was ended halts `loop-exited` at its first pass, as after a crash.
A stop that dies midway leaves its pause entry, which `lym tender resume` lifts.
A standing Bolt hold and a hub pause that stood before the stop are left as they are.
The operator closes terminal windows by hand.

## Tests

- `TestPauseAndConfirm` (hibernateengine, tier 0): the entry the pause adds is owned under the caller's label;
  a standing hub entry is neither owned nor lifted; a busy session through the whole wait times out, naming it.
- `TestRunHubStop` (beamcli, tier 0): the sequence's order, with the prime's session last;
  only an owned entry is lifted; an unconfirmed pause and a failed pair down are named and every later step still runs.
- `TestBeamDown_StopsOrchThenTheDobbyThenTheOrchProcess` (orchcli, tier 0) pins the orch process end as the last step.
- The shipped-buttons test in reedengine pins the new `stop` entry in both templates.
