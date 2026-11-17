# lym machine start: bring back every hub this machine owns

After a crash or reboot the operator re-finds the hubs and beams that were in flight by hand.
`lym machine start [<shortname>]` now starts what this machine owns in one hub, or in every hub `hubregistry.Live` lists, from any directory.
Desktop launchers call the verb, so one click restores the machine's work.

## What changes

- `lym machine start` takes the worktrees from `dobbyengine.OwnedWorktrees` over each hub's prime and starts them through `lym beam start` as a child process in the prime.
  The child's binary is named through `selfexec.LymBinary`.
  - The prime starts with `--all` when this machine owns it; each beam starts by its slug otherwise.
  - A prime or beam another machine owns is skipped, naming that machine.
  - A prime with no recorded owner is skipped and says so; the hub's owned beams still start.
  - It never goes through `lym s`, whose `lym adopt --here` would take over a hub another machine owns.
  - It resumes no run: the orchs and dobbies take the runs up.
- It opens the hub's VS Code workspace file and, per started worktree, a terminal running `lym reed attach`.
  Where no terminal launcher resolves, it prints each worktree's attach command instead.
- A standing hold prints with its machine and subjects, plus the `lym adopt <subjects>` re-run when the hold is this machine's own.
- `internal/termwindow` gains native Windows through `wt.exe`.
  On native Windows the window runs the attach with no shell around it, so the Windows Terminal profile's `closeOnExit` decides whether it closes.
- New `internal/desktoplauncher` writes the launchers: a `.desktop` file in the freedesktop desktop directory on Linux, a `.cmd` file on the Windows desktop.
  Under WSL the `.cmd` runs the verb through `wsl.exe`.
  There is one launcher per hub and one for every hub; a beam gets none.
- Launcher lifecycle:
  - A dobby start adds the hub's launcher, and the all-hubs one with the first hub, when this machine owns the dobby's worktree, a beam under another machine's prime included.
  - Every catch-up, the dobby start's included, removes the hub's launcher once this machine owns nothing there.
    The catch-up logs a failed removal and fails nothing.
  - The verb adds and removes launchers itself, and removes each on-disk launcher whose hub `hubregistry.Live` no longer lists.
    The all-hubs launcher goes with the last hub.
  - Launcher code lives in `internal/dobbyengine` and the verb, never in `internal/beamcli`.

## Design decisions

- Safety rests on never claiming a prime.
  `beam start` gains a hidden `--no-prime-claim` flag, carried by `orchcli.BeamUpOptions.NoPrimeClaim`, which leaves the prime claim out of `Prepare`.
  The verb passes it for a beam it starts under an unowned prime.
  The flag name is the single exported constant `beamcli.NoPrimeClaimFlag`, which `machinecli` uses to build its child arguments.
- A launcher's removal waits until this machine next runs the verb or a dobby, since no other machine reaches this desktop.
- The verb writes no tracked record.
- `internal/termwindow`'s launchers now take the lym path rather than a prebuilt shell line, so each builds its own attach command.
- Docs: `docs/overview.md`, the `machinecli`, `dobbyengine` and `orchcli` package docs, and `PATTERN-cli-cobra` and `PATTERN-test-isolation` are updated.

## Check by hand

Nothing here ran on a real desktop.

- Run `lym machine start` on Linux, on WSL and on native Windows, and confirm the workspace opens and each terminal attaches.
- Click a generated launcher on each platform and confirm it runs the verb.
- Confirm the hidden `--no-prime-claim` flag name is spelled only through `beamcli.NoPrimeClaimFlag`, with no repeated literal in `internal/machinecli`.
  The one literal left is in the `internal/machinecli/doc.go` prose.
