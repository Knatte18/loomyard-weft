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

- The launcher brings back a whole project: the hub prime's orch, and the orch of every beam under it that was live, each in its terminal window, as starting it by hand does today.
  The orchs and dobbies then take up the runs; the launcher resumes no run itself.
- A `lym` verb does the work, so it also runs from a terminal; the desktop launcher only calls it.
- lym keeps the launchers current as hubs and beams come and go, so a crash finds them already there.

- A launcher starts only what this machine still owns.
  One hub, such as loomyard, is worked on from several machines:
  when OSL-1046 crashes, the operator often goes on from Neve, which takes the hub over,
  so starting OSL-1046 again must not bring that hub's prime or beams up there.
  The verb reads the owner of the hub's prime and of each beam, the owner `lym adopt` writes,
  and skips every one another machine owns, saying which machine holds it.

Fixed by the operator: one launcher per hub plus one that starts every hub;
beams are included; resuming interrupted runs is left to the orchs;
nothing another machine owns is started.

Open: whether it also opens the reed windows of running tasks; `.sh` scripts or `.desktop` launchers on KDE.
