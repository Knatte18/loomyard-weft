# Measuring per-package gate times in a darn writer session

The writer prompt says to run no `lym gate test`, bar one test class through `--gate darn-write`,
while the entry asks for one `lym gate test --tier <n> <pkg>` call per package and tier.
I followed the entry (315 calls with `--wait --no-carry`, run in loops, about 70 minutes) and did not ask the parent first.
A measurement entry needs the prompt to say that its own gate calls are allowed.

The call's wall time includes the wait for a gate slot (up to 300 s here),
so wall figures had to come from each run record's `wall_seconds`.
