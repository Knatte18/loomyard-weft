# Board

Rendered 2026-11-26 07:33 UTC; every age below is as of then.
Prime owner: `neve`.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## [P2 Ideas borrowed from comparable agent orchestrators](project-borrowed-orchestrator-ideas.md) (1)

[`borrowed-orchestrator-ideas`](design-borrowed-orchestrator-ideas.md) · Age 8d · Owner `neve`

### Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F17 |  | **Remaining leads from the survey of comparable agent orchestrators (Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 5w<br>2d | burler, shed, orch, quarry, report |

## [P4 Single-repository hubs: records and board under refs/lym/, the layout known only to fabric](project-single-repo-hub.md) (1)

[`single-repo-hub`](design-single-repo-hub.md) · Age 7d · Owner `neve`

### Research (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R1 |  | **Study whether the hardener finds and fixes real faults**<br>• Judge the two fabric hardener campaigns from their records: real findings, sound fixes, focus kept, cost | [`hardener-works-study`](design-hardener-works-study.md) | 3d<br>2d | burler, fabric |

## [P0 Root: entries under no project](project-root.md) (20)

Owner `neve`

### Running (2)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T1 | **Reed's RC click reads the most recent client's session, not the clicker's** | [`rc-click-wrong-client-session`](design-rc-click-wrong-client-session.md) | darn · light | Darn-Write | neve | 2d<br>2d | reed |
| T10 | **Run transitions commit locally; a per-worktree pusher brings them to origin** | [`async-run-pusher`](design-async-run-pusher.md)<br>[Sketch](decision-async-run-pusher.md) | tapestry · solid | Code-Burler-Fix | neve | 2d<br>2d | batten, fabric, loom, landing, cli, dobby, process, tender, pattern, heddle |

### Layer A (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T8 | **A killed batten cancel leaves a remove record whose way forward fails once the pair is gone**<br>• The record names lym fabric remove --remote, which answers pair not found; batten cancel --remove is the re-run that works | [`remove-record-rerun-pair-gone`](design-remove-record-rerun-pair-gone.md) | darn · light | T10 | 2d<br>2d |  |

### Layer B (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | **Nest loom and tapestry config keys by area, move old names on reconcile, and name every key in full** | [`config-keys-nested`](design-config-keys-nested.md)<br>[Sketch](decision-config-keys-nested.md) | tapestry · solid | T1<br>T10<br>T8 | 2d<br>2d | config, loom, reconcile, profiles, shed, batten, fabric, drift, landing, gate, webster, shuttle, dobby, claude, codex, stencil, pattern, docs |

### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 2w<br>2w | gate, fabric |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 8d<br>8d | shuttle |

### Bugs (11)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **No tmux-tier test sends parent-review ask from a live non-orch pane** | [`pr-ask-live-pane-test`](design-pr-ask-live-pane-test.md) | 2d<br>2d | landing, creel |
| B2 |  | **reed switch and a button's {strand} read the most recent client's session, not the clicker's**<br>• Same display-message -c with no -t defect the RC click had, in switch.go and buttonclick.go | [`reed-switch-wrong-client`](design-reed-switch-wrong-client.md) | 2d<br>2d |  |
| B3 |  | **tender revive reads Claude Code placeholder after an API error as a typed draft and skips the session** | [`revive-reads-placeholder-draft`](design-revive-reads-placeholder-draft.md) | 8m<br>8m | tender, claude |
| B4 |  | **Gate fixer interrupted by a machine stop is reported as dead (timeout)** | [`gate-fixer-reboot-reads-timeout`](design-gate-fixer-reboot-reads-timeout.md) | 0m<br>0m | gate, shuttle |
| B5 |  | **creel post refusal after a persisted cd does not say why** | [`creel-refusal-after-cd`](design-creel-refusal-after-cd.md) | 0m<br>0m | creel, cwd |
| B6 |  | **Describe halts failed on an uninformative shuttle run outcome timeout** | [`describe-timeout-no-detail`](design-describe-timeout-no-detail.md) | 0m<br>0m | landing, shuttle |
| B7 |  | **Unreachable origin halts the run as failed repeatedly instead of one retryable stop** | [`unreachable-origin-halts-failed`](design-unreachable-origin-halts-failed.md) | 0m<br>0m | shed, landing, fabric |
| B8 |  | **Finalize blocks on untracked leftovers of a project the merge-in deleted** | [`finalize-ignored-leftover-dir`](design-finalize-ignored-leftover-dir.md) | 0m<br>0m | landing |
| B9 |  | **Describe seat waits out the full timeout after an operator interrupt, then halts as hard failure** | [`describe-held-turn-waits-timeout`](design-describe-held-turn-waits-timeout.md) | 0m<br>0m | landing, shuttle |
| B10 |  | **Finalize halts as hard failure on a GitHub API EOF that lym marks transient** | [`finalize-transient-eof-fails`](design-finalize-transient-eof-fails.md) | 0m<br>0m | landing, shed |
| B11 |  | **A died bouncer judge session spends Code-Bouncer's bounce budget** | [`died-judge-spends-bounce-budget`](design-died-judge-spends-bounce-budget.md) | 0m<br>0m | burler, shed |

### Enhancements (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **Periodic async push of pairs whatever made the commits** | [`periodic-pair-push`](design-periodic-pair-push.md) | 0m<br>0m | fabric, dobby |
| E2 |  | **Iterative agents should commit often** | [`writers-commit-each-step`](design-writers-commit-each-step.md) | 0m<br>0m | stencil, darn |

### Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 3w<br>2d | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
