# A closed area per hub: Landlock and a seccomp connect supervisor

`closed-area` · Project · feature, creel, cwd, reed, shuttle, claude, codex, dobby, slack, process, git, cli, docs, pattern, termwindow, hibernate, batten, ide, deploy

Decision record: [Sketch](decision-closed-area.md)

Groups the entries filed from the Sketch `sandbox-closed-area-build`: area-creel-identity, area-seccomp-core, area-apparmor-barrier, area-host-exits.

## Parked

Not to be released or worked yet (operator, 2026-11-26): there is far too little data to build it, and the right kernel is missing.
The kernel must be upgraded well beyond what the machines run before any of it can work: the Landlock version in use has no socket support (operator, 2026-11-26).
Its tasks stay draft until the operator takes it up again.
The closed area is built last of all, with no other run in flight (operator, 2026-11-26); this holds for every member, area-creel-identity included, though it has no blocker on the board.
