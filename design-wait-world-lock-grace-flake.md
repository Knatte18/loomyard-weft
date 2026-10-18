# shedverbs wait-world integration tests flake: loop exited before it recorded itself

`wait-world-lock-grace-flake` · Task · bug, shed · running · Describe

Imported from [issue #716](https://github.com/Knatte18/loomyard/issues/716).

Two consecutive Merge-In verifies failed in the `internal/shedverbs` integration tier, on different wait-world tests.
Each failure had the waiter reporting `loop … exited before it recorded itself`.
The loom run halted blocked at Merge-In (`escalation-to-human`) with the way forward `lyx loom goto --to Darn-Write`, then `lyx loom resume`.

Why it is abnormal: the failures did not reproduce in isolation, under `-race`, under `-count=300` or beside heavy integration packages, so diagnosing them cost several runs.
The likely cause is the wait-world fixture's 20 ms `LockGrace`, which the waiter's free-lock probe can exhaust under scheduling stalls.

What to do: give the wait-world fixture the production lock grace (the task's fix did this), and check other fixtures in the integration tier for short grace values that make verify nondeterministic.

Provenance: task `gate-wait-cache-cap`, notes `Darn-Write.md`, `loom-halt.md`.
