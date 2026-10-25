# shuttle start fails on transient tmux fork/exec EAGAIN and leaves the strand behind

`shuttle-retry-tmux-eagain` · Note · bug, shuttle, reed

Imported from [issue #869](https://github.com/Knatte18/loomyard/issues/869).

What happened: a `lyx loom` run halted as `failed` (anomaly `producer-hard-failure`) at the `Darn-Write` seat producer.
The shuttle start reported "the provider never became ready" with outcome "died", then could not remove the strand because the liveness check failed with `fork/exec /usr/bin/tmux: resource temporarily unavailable`.
The error chain ended in "deliver the prompt after loading skills: check strand liveness: check session".
The halt message told the operator to remove the strand by hand with `lyx reed status` / `lyx reed remove`.

Why it is abnormal: `resource temporarily unavailable` (EAGAIN) on fork/exec is a transient host condition, yet it was treated as the provider dying.
The first tmux failure appears to have been read as a dead provider, so the run hard-failed on a recoverable condition, and the cleanup hit the same error and left an orphan strand.

What to do: retry tmux invocations on EAGAIN with a short backoff in the shuttle liveness and removal paths, and do not classify an unverifiable liveness check as "died".
Also retry or defer strand removal so a failed start does not leave a strand needing manual cleanup.

Provenance: task `webster-reset-reopens-dead-batch`, note `loom-halt.md`.
