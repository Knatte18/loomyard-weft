# One desktop launcher brings back every hub and beam that was in flight after a crash or reboot

`machine-restart-shortcut` · Note · feature, orch, reed, termwindow, shortcut

## Seen (operator, 2026-11-08 and 2026-11-11)

OSL-1046 and Neve crash often.
After a crash, bringing lym back takes the operator several manual steps:
finding every hub and project that was in flight, often three or four, with beams under them,
and starting each orch session by hand.
Once an orch is up, it and its dobby take the runs up again by themselves;
a run whose step was cut off halts (`loop-exited`) and the orch settles it.
Finding what was running is the hard part, not resuming it.
VS Code needs nothing: on Kubuntu it reopens the windows it had open by itself.

## Idea (operator)

A simple way to start again every project that was in flight on the machine, such as launcher scripts lym puts on the desktop:

- The launcher brings back a whole project: the hub prime and every beam under it that was live.
  It opens the hub's one VS Code workspace file, `<hub>/<repo>.code-workspace`, which lists every worktree of the hub
  (note `hub-workspace-all-pairs` makes it list the pairs under beams too).
  For the prime and each such beam it opens a terminal outside VS Code in the worktree running `lym s`,
  which starts the worktree's dobby and orch and attaches to its reed session.
  The operator never types `lym reed attach`; a running task's window opens from the orch or the Reed status bar's terminal button, so the launcher opens none.
  The orchs and dobbies then take up the runs; the launcher resumes no run itself.
- A `lym` verb does the work, so it also runs from a terminal; the desktop launcher only calls it.
- The launcher is the form the machine's desktop runs: a `.desktop` file on Linux (the freedesktop standard, so KDE, GNOME and others), a `.cmd` file on Windows.
  Both call the same `lym` verb.
- lym itself adds and removes the launchers as hubs and beams come and go,
  and removes a hub's launcher when another machine takes the hub over,
  so a crash finds the right set already on the desktop; the operator never makes or deletes one by hand.

- A launcher starts only what this machine still owns.
  One hub, such as loomyard, is worked on from several machines:
  when OSL-1046 crashes, the operator often goes on from Neve, which takes the hub over,
  so starting OSL-1046 again must not bring that hub's prime or beams up there.
  The verb reads the owner of the hub's prime and of each beam, the owner `lym adopt` writes,
  and skips every one another machine owns, saying which machine holds it.

Fixed by the operator: one launcher per hub plus one that starts every hub;
beams are included; resuming interrupted runs is left to the orchs;
nothing another machine owns is started; `.desktop` on Linux and `.cmd` on Windows, added and removed by lym.
