# Darn-Write: tracing #897 and gating the creel change

The board entry's suspected cause, a race with the `asyncRewake` Stop waiter, was wrong: the orch session ran settings from before that hook's deploy.
Only the orch's Claude transcript showed the `/compact` had landed, and the creel daemon writes no delivery-level log, so the shuttle error behind the `unconfirmed` record had to be inferred from timing.

`TestCreelCLI`'s EXTERNAL row failed twice in full-package tier1 runs while other worktrees' gates loaded the machine.
It then passed alone, three times as a whole test, twice as a whole package and in the final `--upto 2` run.
Telling a flake from a regression took a stash round-trip to the base tree.
