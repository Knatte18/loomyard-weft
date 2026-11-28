# P7 A closed area per hub: Landlock and a seccomp connect supervisor

Rendered 2026-11-28 12:52 UTC; every age below is as of then.
Under: [Board](README.md).

[`closed-area`](design-closed-area.md) · Age 2d · Owner `neve`

Groups the entries filed from the Sketch `sandbox-closed-area-build`: area-creel-identity, area-seccomp-core, area-apparmor-barrier, area-host-exits.

## Parked

Not to be released or worked yet (operator, 2026-11-26): there is far too little data to build it, and the right kernel is missing.
The kernel must be upgraded well beyond what the machines run before any of it can work: the Landlock version in use has no socket support (operator, 2026-11-26).
Its tasks stay draft until the operator takes it up again.
The closed area is built last of all, with no other run in flight (operator, 2026-11-26); this holds for every member, area-creel-identity included, though it has no blocker on the board.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Draft (4)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T7 | `draft` **creel across an area: kernel-attached sender identity, the area registry and the hub directory** | [`area-creel-identity`](design-area-creel-identity.md)<br>[Sketch](decision-area-creel-identity.md) | darn · solid |  | 2d<br>2d | creel, cwd |
| T14 | `draft` **Stage 1 of the closed area: supervisor, Landlock ruleset, connect filter, attach client and the in-area engines, token and relay** | [`area-seccomp-core`](design-area-seccomp-core.md)<br>[Sketch](decision-area-seccomp-core.md) | tapestry · solid | T7 | 2d<br>2d | reed, shuttle, claude, codex, creel, dobby, slack, process, cwd, git, cli, docs, pattern |
| T15 | `draft` **The closed area's AppArmor fallback barrier, built only when the OSL-1046 probe fails** | [`area-apparmor-barrier`](design-area-apparmor-barrier.md)<br>[Sketch](decision-area-apparmor-barrier.md) | darn · solid | T14 | 2d<br>2d | process |
| T16 | `draft` **Stage 2 of the closed area: window, hibernate and deploy as supervisor verbs, and lym ide refused inside** | [`area-host-exits`](design-area-host-exits.md)<br>[Sketch](decision-area-host-exits.md) | darn · solid | T15 | 2d<br>2d | termwindow, hibernate, batten, ide, deploy, reed |
