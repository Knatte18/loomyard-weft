# P6 Intake motifs: run watch, landed records, closed area

Rendered 2026-11-26 12:44 UTC; every age below is as of then.
Under: [Board](README.md).

`intake-motifs` · Age 4h · Owner `neve`

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## [P7 A closed area per hub: Landlock and a seccomp connect supervisor](project-closed-area.md) (4)

[`closed-area`](design-closed-area.md) · Age 45m · Owner `neve`

### Draft (4)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T7 | `draft` **creel across an area: kernel-attached sender identity, the area registry and the hub directory** | [`area-creel-identity`](design-area-creel-identity.md)<br>[Sketch](decision-area-creel-identity.md) | darn · solid | T3 | 45m<br>45m | creel, cwd |
| T14 | `draft` **Stage 1 of the closed area: supervisor, Landlock ruleset, connect filter, attach client and the in-area engines, token and relay** | [`area-seccomp-core`](design-area-seccomp-core.md)<br>[Sketch](decision-area-seccomp-core.md) | tapestry · solid | T7<br>T3<br>T10<br>T8<br>T11 | 45m<br>40m | reed, shuttle, claude, codex, creel, dobby, slack, process, cwd, git, cli, docs, pattern |
| T15 | `draft` **The closed area's AppArmor fallback barrier, built only when the OSL-1046 probe fails** | [`area-apparmor-barrier`](design-area-apparmor-barrier.md)<br>[Sketch](decision-area-apparmor-barrier.md) | darn · solid | T14 | 45m<br>45m | process |
| T16 | `draft` **Stage 2 of the closed area: window, hibernate and deploy as supervisor verbs, and lym ide refused inside** | [`area-host-exits`](design-area-host-exits.md)<br>[Sketch](decision-area-host-exits.md) | darn · solid | T15 | 45m<br>45m | termwindow, hibernate, batten, ide, deploy, reed |

## Own entries

### Draft (2)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T8 | `draft` **Dobby run watch and gate-hold notice: mail the parent on every stop and on a slot held too long** | [`dobby-run-watch-build`](design-dobby-run-watch-build.md)<br>[Sketch](decision-dobby-run-watch-build.md) | tapestry · solid | T3<br>T4 | 1h<br>1h | dobby, gate, batten, orch, pattern |
| T17 | `draft` **Keep each landed run's findings, asks and fixer reports in its archive tag, with an index row and lym report landed** | [`keep-landed-run-records`](design-keep-landed-run-records.md)<br>[Sketch](decision-keep-landed-run-records.md) | tapestry · solid | T3<br>T1<br>T9<br>T11<br>T12<br>T13 | 45m<br>41m | landing, fabric, report, loom, burler, shed, cwd, git, testkit, pattern |
