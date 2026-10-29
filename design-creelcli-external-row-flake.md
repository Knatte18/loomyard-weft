# TestCreelCLI EXTERNAL row flakes in full-package tier1 runs under machine load

`creelcli-external-row-flake` · Note · bug, creel

Imported from [issue #901](https://github.com/Knatte18/loomyard/issues/901).

The `TestCreelCLI` EXTERNAL row failed twice in full-package tier1 runs while other worktrees' gates loaded the machine.
It then passed alone, three times as a whole test, twice as a whole package, and in the final `lym gate test --upto 2` run.

This is abnormal because the failure did not reproduce without load, so it looks like a timing-sensitive test rather than a regression.
Telling the two apart took a stash round-trip to the base tree, which is costly for an agent that cannot tell a flake from a break.

What to do: find the timing assumption in the EXTERNAL row (a fixed wait or a tight deadline) and make it load-tolerant; if the gate can tell a load-induced failure from a real one, have it say so.

Provenance: task `compact-inject-draft`, note `Darn-Write.md`.
