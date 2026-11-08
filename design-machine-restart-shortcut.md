# One desktop shortcut brings every lym session of the machine back after a crash or reboot

`machine-restart-shortcut` · Note · feature, orch, reed, termwindow, shortcut

## Seen (operator, 2026-11-08)

Neve crashed at about 15:30 and rebooted.
Bringing lym back took the operator several manual steps: starting the hub's orch session by hand, after which the orch and dobby took the runs up again, one by one.
A run whose step was cut off halts (`slack-presence-auto` at Darn-Write, `loop-exited`) and waits for the orch.
VS Code needs nothing: on Kubuntu it reopens the windows it had open by itself.

## Idea (operator, 2026-11-08)

One desktop shortcut brings every lym session of the machine back after a crash or reboot:

- For each hub that was live on the machine, it starts the prime's orch session in its terminal window, as starting it by hand does today; the orch and dobby then take up the runs.
- It opens the reed terminal windows the operator had open, such as the prime's and each running task's.
- The shortcut is a `.desktop` launcher lym installs, which runs one `lym` verb; the verb is the part that does the work, so it can also be run from a terminal.

Open: whether the verb finds the hubs from the machine's hub registry or from a list the operator keeps; whether it opens a reed window for every running task or only the ones that were open; whether interrupted runs are resumed by the verb or left to the orch.
