# Board

Rendered 2026-11-26 08:19 UTC; every age below is as of then.
Prime owner: `neve`.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## [P1 Intake wave 1: PR modes, Webster, dotnet gate, reed, creel, commits](project-intake-wave-1.md) (4)

`intake-wave-1` · Age 0m · Owner `neve`

### Draft (4)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T1 | `draft` **PR block in three modes, review, trail and off, with no agent under it** | [`pr-modes-no-agent`](design-pr-modes-no-agent.md) | darn · solid | T3 | 2d<br>0m | landing, loom, config |
| T2 | `draft` **Webster: plan fixes after begun batches, and recovery that ends asking** | [`webster-plan-recovery`](design-webster-plan-recovery.md) | darn · light | T3 | 45m<br>0m | webster, quarry |
| T4 | `draft` **Gate test fixes for dotnet hubs: hung testhost, loggers, filters, tiers** | [`dotnet-gate-fixes`](design-dotnet-gate-fixes.md) | darn · light | T3 | 46m<br>0m | gate, dotnet, darn, burler |
| T5 | `draft` **Reed reads each client's own session, and shellguard refuses the session bus** | [`reed-client-and-bus-guard`](design-reed-client-and-bus-guard.md) | darn · light | T3 | 2d<br>0m | reed, process |

## [P2 Ideas borrowed from comparable agent orchestrators](project-borrowed-orchestrator-ideas.md) (1)

[`borrowed-orchestrator-ideas`](design-borrowed-orchestrator-ideas.md) · Age 8d · Owner `neve`

### Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F17 |  | **Remaining leads from the survey of comparable agent orchestrators (Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 5w<br>18m | burler, shed, orch, quarry, report |

## [P3 Intake wave 2: Describe text, infra stops, push retries](project-intake-wave-2.md) (0)

`intake-wave-2` · Age 0m · Owner `neve`

## [P4 Single-repository hubs: records and board under refs/lym/, the layout known only to fabric](project-single-repo-hub.md) (1)

[`single-repo-hub`](design-single-repo-hub.md) · Age 7d · Owner `neve`

### Research (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R1 |  | **Study whether the hardener finds and fixes real faults**<br>• Judge the two fabric hardener campaigns from their records: real findings, sound fixes, focus kept, cost | [`hardener-works-study`](design-hardener-works-study.md) | 3d<br>2d | burler, fabric |

## [P5 Intake wave 3: beam orch parent, scope and earlier checks](project-intake-wave-3.md) (0)

`intake-wave-3` · Age 0m · Owner `neve`

## [P6 Intake motifs: run watch, landed records, closed area](project-intake-motifs.md) (0)

`intake-motifs` · Age 0m · Owner `neve`

## [P0 Root: entries under no project](project-root.md) (28)

Owner `neve`

### Running (1)

| # | Task | Slug | Recipe | At | Owner | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- | --- |
| T8 | **A killed batten cancel leaves a remove record whose way forward fails once the pair is gone**<br>• The record names lym fabric remove --remote, which answers pair not found; batten cancel --remove is the re-run that works | [`remove-record-rerun-pair-gone`](design-remove-record-rerun-pair-gone.md) | darn · light | Darn-Write | neve | 2d<br>2d |  |

### Layer A (1)

| # | Task | Slug | Recipe | After | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- | --- |
| T3 | **Nest loom and tapestry config keys by area, move old names on reconcile, and name every key in full** | [`config-keys-nested`](design-config-keys-nested.md)<br>[Sketch](decision-config-keys-nested.md) | tapestry · solid | T8 | 2d<br>2d | config, loom, reconcile, profiles, shed, batten, fabric, drift, landing, gate, webster, shuttle, dobby, claude, codex, stencil, pattern, docs |

### Research (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R3 |  | **Measure what Cortex XDR costs the test suites on OSL-1046**<br>• The 242 s fabricengine tier 1 against 37 to 41 s elsewhere is a machine difference no report has split | [`cortex-cost-measure`](design-cortex-cost-measure.md) | 2w<br>2w | gate, fabric |
| R4 |  | **Antigravity CLI engine spike**<br>• Hands-on spike of agy in a tmux pane: screens, hooks, transcript, login | [`antigravity-engine-spike`](design-antigravity-engine-spike.md) | 8d<br>8d | shuttle |

### Bugs (16)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B3 |  | **tender revive reads Claude Code placeholder after an API error as a typed draft and skips the session** | [`revive-reads-placeholder-draft`](design-revive-reads-placeholder-draft.md) | 54m<br>54m | tender, claude |
| B4 |  | **Gate fixer interrupted by a machine stop is reported as dead (timeout)** | [`gate-fixer-reboot-reads-timeout`](design-gate-fixer-reboot-reads-timeout.md) | 46m<br>46m | gate, shuttle |
| B5 |  | **creel post refusal after a persisted cd does not say why** | [`creel-refusal-after-cd`](design-creel-refusal-after-cd.md) | 46m<br>46m | creel, cwd |
| B6 |  | **Describe halts failed on an uninformative shuttle run outcome timeout** | [`describe-timeout-no-detail`](design-describe-timeout-no-detail.md) | 46m<br>46m | landing, shuttle |
| B7 |  | **Unreachable origin halts the run as failed repeatedly instead of one retryable stop** | [`unreachable-origin-halts-failed`](design-unreachable-origin-halts-failed.md) | 46m<br>46m | shed, landing, fabric |
| B8 |  | **Finalize blocks on untracked leftovers of a project the merge-in deleted** | [`finalize-ignored-leftover-dir`](design-finalize-ignored-leftover-dir.md) | 46m<br>46m | landing |
| B9 |  | **Describe seat waits out the full timeout after an operator interrupt, then halts as hard failure** | [`describe-held-turn-waits-timeout`](design-describe-held-turn-waits-timeout.md) | 46m<br>46m | landing, shuttle |
| B10 |  | **Finalize halts as hard failure on a GitHub API EOF that lym marks transient** | [`finalize-transient-eof-fails`](design-finalize-transient-eof-fails.md) | 46m<br>46m | landing, shed |
| B11 |  | **A died bouncer judge session spends Code-Bouncer's bounce budget** | [`died-judge-spends-bounce-budget`](design-died-judge-spends-bounce-budget.md) | 46m<br>46m | burler, shed |
| B12 |  | **Transient push failures are mailed to the orch one by one instead of retried** | [`push-failures-mailed-one-by-one`](design-push-failures-mailed-one-by-one.md) | 46m<br>46m | fabric |
| B14 |  | **Beam orch cannot amend its own beam's decision record** | [`beam-orch-cannot-amend-record`](design-beam-orch-cannot-amend-record.md) | 46m<br>46m | loom, orch, pawl |
| B18 |  | **A beam member's run records the prime orch as its parent orch** | [`beam-member-parent-is-prime`](design-beam-member-parent-is-prime.md) | 45m<br>45m | batten, landing, orch |
| B19 |  | **Tackler and driver prompts of a beam member say no parent is recorded** | [`beam-member-prompt-no-parent`](design-beam-member-prompt-no-parent.md) | 45m<br>45m | stencil, shed |
| B21 |  | **The GitHub PR body is not updated when the description file changes** | [`pr-body-not-synced`](design-pr-body-not-synced.md) | 45m<br>45m | landing |
| B24 |  | **Shuttle idle check reports no input box on an idle seat, killing Plan-Burler-Review** | [`seat-idle-no-input-box`](design-seat-idle-no-input-box.md) | 45m<br>45m | shuttle, claude |
| B25 |  | **Seed-IDE fails the run when the task branch's detached push has not landed yet** | [`seed-ide-push-not-landed`](design-seed-ide-push-not-landed.md) | 45m<br>45m | batten, fabric |

### Enhancements (6)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **Periodic async push of pairs whatever made the commits** | [`periodic-pair-push`](design-periodic-pair-push.md) | 46m<br>46m | fabric, dobby |
| E2 |  | **Iterative agents should commit often** | [`writers-commit-each-step`](design-writers-commit-each-step.md) | 46m<br>46m | stencil, darn |
| E3 |  | **Keep the decision record and review findings of each landed task in the weft** | [`keep-landed-records-in-weft`](design-keep-landed-records-in-weft.md) | 45m<br>45m | landing, fabric, report |
| E4 |  | **The Describe text names no LoomYard terms, except the Created using block** | [`describe-no-loomyard-terms`](design-describe-no-loomyard-terms.md) | 45m<br>45m | landing, stencil |
| E5 |  | **Close the scope of a task when its first Code-Review round starts, and have lym hold the line** | [`scope-closes-at-code-review`](design-scope-closes-at-code-review.md) | 45m<br>45m | loom, burler |
| E6 |  | **Review the Describe text inside the Describe row, and catch the rest of what the parent finds at PR-Review earlier** | [`review-describe-text`](design-review-describe-text.md) | 45m<br>45m | landing, burler |

### Features (2)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F2 |  | **Built-in run watch: mail the parent on every non-running state and on a gate slot held too long** | [`dobby-run-watch`](design-dobby-run-watch.md) | 45m<br>45m | dobby, gate |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 3w<br>9m | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |
