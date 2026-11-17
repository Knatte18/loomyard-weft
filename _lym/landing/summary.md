# lym machine start brings back this machine's hubs from desktop launchers

After a crash or a machine stop, the operator had to find the hubs and beams that were in flight and start each by hand.
`lym machine start [<shortname>]` now does that from any directory, and desktop launchers run it with one click.

## What changes

- **`lym machine start`** (`internal/machinecli/start.go`) walks every hub `hubregistry.Live` lists, or the one hub `hubregistry.Lookup` resolves for the shortname.
  Per hub it pulls the board, then sorts the worktrees `dobbyengine.OwnedWorktrees` lists over `hubgeom.DobbyStart`, whose owner reads go through the machine registry.
  - The prime owned here starts with `lym beam start --all`, which brings up every owned beam with it.
  - Otherwise each owned beam starts with `lym beam start <slug>`.
  - A prime or beam another machine owns is skipped, naming that machine and `lym adopt`.
  - A prime or beam with no recorded owner is skipped and says so, since `dobbyengine.EnsureLive` would claim it.
  The children run in the hub's prime, named through `selfexec.LymBinary`, logged and waited on, and their envelopes decide each worktree's outcome.
  The verb never runs `lym s` or `lym adopt`, and it resumes no run.
  In a hub where something started it opens the hub's workspace file through `vscode.Launch` and one terminal per started worktree running `lym reed attach`.
  Where no terminal launcher resolves, each started worktree carries its attach command in the result.
  A standing Bolt hold is printed with its machine and subjects, and this machine's own hold also carries the `lym adopt <subjects>` re-run.
  A hub that does not read, or a worktree that does not start, makes the verb exit non-zero, and the other hubs still go on.
- **`internal/desktoplauncher`** (new) locates the desktop and writes the launchers: `lym-hub-<shortname>` per hub and `lym-hubs` for every hub.
  On Linux they are freedesktop `.desktop` files in the `user-dirs.dirs` desktop, else `$HOME/Desktop`.
  Under WSL they are `.cmd` files on the Windows desktop, found through `powershell.exe` and `wslpath`, and they run the verb through `wsl.exe`.
  On native Windows they are `.cmd` files on the desktop's known folder.
  `XDG_DESKTOP_DIR` overrides the directory on every platform.
  A machine with no desktop directory gets no launcher.
  `Remove` takes the all-hubs launcher with the last hub's.
- **Launcher upkeep.**
  - Each dobby start, a re-exec's included, adds the hub's launcher when `Deps.Owner` reads its worktree owned here (`internal/dobbyengine/launcher.go`).
  - `dobbyengine.CatchUp` removes it once `dobbyengine.OwnsAnything` reads that the pulled board records nothing this machine owns: neither the prime nor a beam that is not done.
    The removal is part of `CatchUp` itself, so the dobby start's catch-up, the one after a sleep, a hand pass's and an adopt's all do it.
  - The verb adds or removes each hub's launcher by the same rule.
    It also removes each launcher whose hub `hubregistry.Live` no longer lists.
- **`internal/termwindow`** resolves Windows Terminal on native Windows.
  The window runs `lym reed attach` in the worktree with no shell around it.
  Each launcher's argument builder now takes the lym path and builds its own line.

## Design decisions

- The launcher removal lives in `dobbyengine.CatchUp`, not in the dobby loop, so every catch-up does it, as the record asks.
  `battencli.HubCatchUp`, the one constructor of the catch-up seams, now fills the new `CatchUpDeps.Launcher` with the hub's board directory and recorded shortname.
  That is a two-line change in `internal/battencli`, a package the record's scope does not list.
  `internal/beamcli` is untouched.
  The dobby start reads the same `Launcher` for its add, so a dobby carries the hub's shortname in one place.
- `dobbyengine` imports `internal/desktoplauncher`, `internal/machinename` and `internal/selfexec`, and its import allowlist gains them.
  The board directory and shortname still come from its caller.
  The desktop directory is outside every hub and is named by `desktoplauncher` alone.
- `tmuxkit.Main` now points `XDG_DESKTOP_DIR` at its run directory, as it does `XDG_CONFIG_HOME`.
  Without that, a test process or a dobby a test starts would write launchers on the operator's desktop.
- A beam whose effective owner is empty, which happens only under an unowned prime, is skipped like the unowned prime.
  The record's criterion is that the verb starts only the beams owned here.
  `OwnedWorktrees` lists such a beam, since it reads no other machine.
- On native Windows the attach runs bare in Windows Terminal, so the profile's `closeOnExit` decides whether the window closes.
  The Linux and WSL windows keep their own close-on-exit shell line.
- A launcher terminal on Linux waits for Enter, and a `.cmd` launcher ends in `pause`, so the operator sees the verb's result.

## Bounds

- A launcher outlives this machine's ownership until this machine next runs the verb or a catch-up, since no other machine reaches its desktop.
- The verb writes no tracked record and claims no prime.
