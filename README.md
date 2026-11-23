# Board

Rendered 2026-11-23 18:35 UTC; every age below is as of then.
Prime owner: `neve`.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## [P2 Ideas borrowed from comparable agent orchestrators](project-borrowed-orchestrator-ideas.md) (1)

[`borrowed-orchestrator-ideas`](design-borrowed-orchestrator-ideas.md) · Age 6d · Owner `neve`

### Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F17 |  | **Remaining leads from the survey of comparable agent orchestrators (Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 5w<br>7h | burler, shed, orch, quarry, report |

## [P4 Single-repository hubs: records and board under refs/lym/, the layout known only to fabric](project-single-repo-hub.md) (1)

[`single-repo-hub`](design-single-repo-hub.md) · Age 4d · Owner `neve`

### Research (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R1 |  | **Study whether the hardener finds and fixes real faults**<br>• Judge the two fabric hardener campaigns from their records: real findings, sound fixes, focus kept, cost | [`hardener-works-study`](design-hardener-works-study.md) | 1d<br>11m | burler, fabric |

## [P0 Root: entries under no project](project-root.md) (11)

Owner `neve`

### Running (5)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T2 | **Reed's Slack button click leaves forwarding off with no trace** | [`slack-click-silent-fail`](design-slack-click-silent-fail.md) | darn · light | Code-Burler-Fix | neve | 4h<br>4h | reed, slack |
| T4 | **PR-Review row does not notice its session ended** | [`pr-review-dead-session-unseen`](design-pr-review-dead-session-unseen.md) | darn · light | Code-Burler-Fix | neve | 5h<br>5h | landing, shuttle |
| T5 | **Study: a lym-started Landlock area instead of bwrap** | [`landlock-area-study`](design-landlock-area-study.md) | darn · speed | Darn-Write | neve | 8m<br>8m | shuttle, reed, claude |
| T9 | **Red-first gate must support C# repositories** | [`red-first-csharp`](design-red-first-csharp.md) | darn · solid | Code-Burler-Fix | neve | 5h<br>4h | gate, loom, dotnet |
| T10 | **Run transitions commit locally; a per-worktree pusher brings them to origin** | [`async-run-pusher`](design-async-run-pusher.md)<br>[Sketch](decision-async-run-pusher.md) | tapestry · solid | Plan-Bouncer | neve | 9h<br>9h | batten, fabric, loom, landing, cli, dobby, process, tender, pattern, heddle |

### Layer A (2)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T1 | **Autonomy: a failed later ask hides the parent's satisfied answer, and the parent cannot settle it** | [`failed-ask-hides-satisfied`](design-failed-ask-hides-satisfied.md) | darn · light | T4 | 4h<br>4h | landing, loom |
| T8 | **A killed batten cancel leaves a remove record whose way forward fails once the pair is gone**<br>• The record names lym fabric remove --remote, which answers pair not found; batten cancel --remove is the re-run that works | [`remove-record-rerun-pair-gone`](design-remove-record-rerun-pair-gone.md) | darn · light | T10 | 4h<br>4h |  |

### Layer B (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | **Nest loom and tapestry config keys by area, move old names on reconcile, and name every key in full** | [`config-keys-nested`](design-config-keys-nested.md)<br>[Sketch](decision-config-keys-nested.md) | tapestry · solid | T2<br>T9<br>T10<br>T4<br>T8<br>T1 | 5h<br>4h | config, loom, reconcile, profiles, shed, batten, fabric, drift, landing, gate, webster, shuttle, dobby, claude, codex, stencil, pattern, docs |

### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 2w<br>2w | gate, fabric |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 5d<br>5d | shuttle |

### Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 3w<br>1d | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
