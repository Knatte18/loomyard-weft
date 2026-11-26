# P7 A closed area per hub: Landlock and a seccomp connect supervisor

Rendered 2026-11-26 12:43 UTC; every age below is as of then.
Under: [P6 Intake motifs: run watch, landed records, closed area](project-intake-motifs.md).

[`closed-area`](design-closed-area.md) · Age 44m · Owner `neve`

Groups the entries filed from the Sketch `sandbox-closed-area-build`: area-creel-identity, area-seccomp-core, area-apparmor-barrier, area-host-exits.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## Draft (4)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T7 | `draft` **creel across an area: kernel-attached sender identity, the area registry and the hub directory** | [`area-creel-identity`](design-area-creel-identity.md)<br>[Sketch](decision-area-creel-identity.md) | darn · solid | T3 | 44m<br>44m | creel, cwd |
| T14 | `draft` **Stage 1 of the closed area: supervisor, Landlock ruleset, connect filter, attach client and the in-area engines, token and relay** | [`area-seccomp-core`](design-area-seccomp-core.md)<br>[Sketch](decision-area-seccomp-core.md) | tapestry · solid | T7<br>T3<br>T10<br>T8<br>T11 | 44m<br>39m | reed, shuttle, claude, codex, creel, dobby, slack, process, cwd, git, cli, docs, pattern |
| T15 | `draft` **The closed area's AppArmor fallback barrier, built only when the OSL-1046 probe fails** | [`area-apparmor-barrier`](design-area-apparmor-barrier.md)<br>[Sketch](decision-area-apparmor-barrier.md) | darn · solid | T14 | 44m<br>44m | process |
| T16 | `draft` **Stage 2 of the closed area: window, hibernate and deploy as supervisor verbs, and lym ide refused inside** | [`area-host-exits`](design-area-host-exits.md)<br>[Sketch](decision-area-host-exits.md) | darn · solid | T15 | 44m<br>44m | termwindow, hibernate, batten, ide, deploy, reed |
