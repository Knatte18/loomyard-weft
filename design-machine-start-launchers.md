# lym machine start brings back every hub this machine owns, from desktop launchers

`machine-start-launchers` · Task · feature, termwindow, dobby · running · Darn-Write

Decision record: [Sketch](decision-machine-start-launchers.md)

Depends on: [`leftover-pairs-teardown`](design-leftover-pairs-teardown.md)

Filed from the Sketch `adopt-build-resplit`.

Takes: machinecli's cobra subtree and deps seams; hubregistry.Live and Lookup; dobbyengine.OwnedWorktrees and EnsureLive with their registry-aware owner reads and the catch-up from machine-registry-holds-catchup, run at the dobby start, after a sleep and on a hand pass; the Bolt hold reader; lym beam start --all as a child process named through selfexec.LymBinary; termwindow.Resolve and OpenAttach; fabricengine.HubWorkspacePath and vscode.Launch
Delivers: lym machine start [<shortname>] starting the prime and owned beams of one hub or every live hub through lym beam start in the hub's prime, skipping a prime another machine owns or one with no recorded owner and saying so, never through lym s; the hub's workspace file opened and one terminal per started worktree running lym reed attach, or each attach command printed where no launcher resolves; a standing hold printed with its machine, subjects and, for this machine's own, the lym adopt <subjects> re-run; native Windows (wt.exe) in termwindow; internal/desktoplauncher writing .desktop files on Linux and .cmd files on the Windows desktop, through wsl.exe under WSL; the launcher added at the dobby start and removed by every catch-up, the dobby start's included, that finds this machine owns nothing in the hub, in internal/dobbyengine, and added and removed by the verb, never in internal/beamcli
Exports: internal/desktoplauncher's add and remove functions and machinecli's start verb

The part changes these packages:

- internal/machinecli (about 420 changed lines)
- internal/desktoplauncher (about 380 changed lines)
- internal/termwindow (about 90 changed lines)
- internal/dobbyengine (about 80 changed lines)
