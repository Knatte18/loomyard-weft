# Burler fix round: twill fixer fork attempts write/edit calls and halts the run as producer-hard-failure

`twill-fixer-fork-writes` · Task · bug, burler, lenses · running · Record-Write

Depends on: [`shuttle-start-idle-and-prompt`](design-shuttle-start-idle-and-prompt.md)

Imported from [issue #144](https://github.com/Knatte18/loomyard/issues/144).

What happened: a loom run halted failed at the Code-Burler-Fix producer (anomaly producer-hard-failure) in round 1 attempt 1. The shedadapters guard reported that the burler twill fixer fork attempted 5 write/edit tool calls, and that a fork must never mutate a file. Way forward named by lym: inspect the target's working-tree changes, revert what the fork wrote, and resume the run. The preceding Code-Bouncer and Code-Burler-Review rows were both recorded as stuck.

Why it is abnormal: the fix round is the step meant to apply review findings, yet the fork it spawns is forbidden to write, and the fork tried anyway. Either the fixer fork's brief does not tell it that it is read-only and where to hand its fixes (so it reaches for Edit/Write), or the fix round routes work to a fork that should have been a writing session. Either way an ordinary run ends in a hard failure that needs manual revert-and-resume, and the writes may already have touched the working tree.

What to do: make the Code-Burler-Fix fork's prompt state the no-mutation contract and the channel for returning fixes, or run the fixer as a session allowed to write; and consider having the guard block or revert the fork's writes itself, or retry the attempt once with a corrective message, instead of failing the whole run.

Provenance: task slug leftover-pairs-teardown; note file loom-halt.md.