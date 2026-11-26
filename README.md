# Board

Rendered 2026-11-26 08:06 UTC; every age below is as of then.
Prime owner: `neve`.

Tasks are grouped by dependency layer and notes by type.
An entry waits only on the open entries it names under After, so the entries in one layer can run in parallel.

## [P2 Ideas borrowed from comparable agent orchestrators](project-borrowed-orchestrator-ideas.md) (1)

[`borrowed-orchestrator-ideas`](design-borrowed-orchestrator-ideas.md) · Age 8d · Owner `neve`

### Features (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F17 |  | **Remaining leads from the survey of comparable agent orchestrators (Oct 2026)** | [`orchestrator-survey-ideas`](design-orchestrator-survey-ideas.md) | 5w<br>5m | burler, shed, orch, quarry, report |

## [P4 Single-repository hubs: records and board under refs/lym/, the layout known only to fabric](project-single-repo-hub.md) (1)

[`single-repo-hub`](design-single-repo-hub.md) · Age 7d · Owner `neve`

### Research (1)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| R1 |  | **Study whether the hardener finds and fixes real faults**<br>• Judge the two fabric hardener campaigns from their records: real findings, sound fixes, focus kept, cost | [`hardener-works-study`](design-hardener-works-study.md) | 3d<br>2d | burler, fabric |

## [P0 Root: entries under no project](project-root.md) (41)

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

### Bugs (26)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| B1 |  | **No tmux-tier test sends parent-review ask from a live non-orch pane** | [`pr-ask-live-pane-test`](design-pr-ask-live-pane-test.md) | 2d<br>2d | landing, creel |
| B2 |  | **reed switch and a button's {strand} read the most recent client's session, not the clicker's**<br>• Same display-message -c with no -t defect the RC click had, in switch.go and buttonclick.go | [`reed-switch-wrong-client`](design-reed-switch-wrong-client.md) | 2d<br>2d |  |
| B3 |  | **tender revive reads Claude Code placeholder after an API error as a typed draft and skips the session** | [`revive-reads-placeholder-draft`](design-revive-reads-placeholder-draft.md) | 41m<br>41m | tender, claude |
| B4 |  | **Gate fixer interrupted by a machine stop is reported as dead (timeout)** | [`gate-fixer-reboot-reads-timeout`](design-gate-fixer-reboot-reads-timeout.md) | 34m<br>34m | gate, shuttle |
| B5 |  | **creel post refusal after a persisted cd does not say why** | [`creel-refusal-after-cd`](design-creel-refusal-after-cd.md) | 33m<br>33m | creel, cwd |
| B6 |  | **Describe halts failed on an uninformative shuttle run outcome timeout** | [`describe-timeout-no-detail`](design-describe-timeout-no-detail.md) | 33m<br>33m | landing, shuttle |
| B7 |  | **Unreachable origin halts the run as failed repeatedly instead of one retryable stop** | [`unreachable-origin-halts-failed`](design-unreachable-origin-halts-failed.md) | 33m<br>33m | shed, landing, fabric |
| B8 |  | **Finalize blocks on untracked leftovers of a project the merge-in deleted** | [`finalize-ignored-leftover-dir`](design-finalize-ignored-leftover-dir.md) | 33m<br>33m | landing |
| B9 |  | **Describe seat waits out the full timeout after an operator interrupt, then halts as hard failure** | [`describe-held-turn-waits-timeout`](design-describe-held-turn-waits-timeout.md) | 33m<br>33m | landing, shuttle |
| B10 |  | **Finalize halts as hard failure on a GitHub API EOF that lym marks transient** | [`finalize-transient-eof-fails`](design-finalize-transient-eof-fails.md) | 33m<br>33m | landing, shed |
| B11 |  | **A died bouncer judge session spends Code-Bouncer's bounce budget** | [`died-judge-spends-bounce-budget`](design-died-judge-spends-bounce-budget.md) | 33m<br>33m | burler, shed |
| B12 |  | **Transient push failures are mailed to the orch one by one instead of retried** | [`push-failures-mailed-one-by-one`](design-push-failures-mailed-one-by-one.md) | 33m<br>33m | fabric |
| B13 |  | **lym gate test answers that the hub declares no tier0, leaving a Burler fix round without a gate** | [`gate-no-tier0-fix-round`](design-gate-no-tier0-fix-round.md) | 33m<br>33m | gate, burler |
| B14 |  | **Beam orch cannot amend its own beam's decision record** | [`beam-orch-cannot-amend-record`](design-beam-orch-cannot-amend-record.md) | 33m<br>33m | loom, orch, pawl |
| B15 |  | **gate test: reports 'ran no test' when the detailed console logger's summary is used** | [`gate-detailed-logger-no-test`](design-gate-detailed-logger-no-test.md) | 33m<br>33m | gate, dotnet |
| B16 |  | **gate test: tier3 command already carries --filter, so a second --filter in args makes dotnet test refuse** | [`gate-tier3-double-filter`](design-gate-tier3-double-filter.md) | 33m<br>33m | gate, dotnet |
| B17 |  | **gate test: darn-write gate key runs tier0 only, so a tier3 measurement test runs nothing** | [`darn-gate-key-tier0-only`](design-darn-gate-key-tier0-only.md) | 33m<br>33m | gate, darn |
| B18 |  | **A beam member's run records the prime orch as its parent orch** | [`beam-member-parent-is-prime`](design-beam-member-parent-is-prime.md) | 33m<br>33m | batten, landing, orch |
| B19 |  | **Tackler and driver prompts of a beam member say no parent is recorded** | [`beam-member-prompt-no-parent`](design-beam-member-prompt-no-parent.md) | 33m<br>33m | stencil, shed |
| B20 |  | **fabric clone leaves the prime without rendered heddle** | [`clone-prime-no-heddle`](design-clone-prime-no-heddle.md) | 32m<br>32m | fabric, heddle |
| B21 |  | **The GitHub PR body is not updated when the description file changes** | [`pr-body-not-synced`](design-pr-body-not-synced.md) | 32m<br>32m | landing |
| B22 |  | **Plan fixes after begun batches block Webster; glyph-not-found on members a card creates** | [`webster-plan-fix-begun-batch`](design-webster-plan-fix-begun-batch.md) | 32m<br>32m | webster, quarry |
| B23 |  | **Webster recovery ending dead with dead_reason asking drops the question and needs a manual reset** | [`webster-recovery-asking-dead`](design-webster-recovery-asking-dead.md) | 32m<br>32m | webster |
| B24 |  | **Shuttle idle check reports no input box on an idle seat, killing Plan-Burler-Review** | [`seat-idle-no-input-box`](design-seat-idle-no-input-box.md) | 32m<br>32m | shuttle, claude |
| B25 |  | **Seed-IDE fails the run when the task branch's detached push has not landed yet** | [`seed-ide-push-not-landed`](design-seed-ide-push-not-landed.md) | 32m<br>32m | batten, fabric |
| B26 |  | **Gate tier timeout_min does not stop a hung dotnet testhost; the slot stayed held 21 hours** | [`gate-timeout-hung-testhost`](design-gate-timeout-hung-testhost.md) | 32m<br>32m | gate, dotnet |

### Enhancements (8)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| E1 |  | **Periodic async push of pairs whatever made the commits** | [`periodic-pair-push`](design-periodic-pair-push.md) | 33m<br>33m | fabric, dobby |
| E2 |  | **Iterative agents should commit often** | [`writers-commit-each-step`](design-writers-commit-each-step.md) | 33m<br>33m | stencil, darn |
| E3 |  | **Keep the decision record and review findings of each landed task in the weft** | [`keep-landed-records-in-weft`](design-keep-landed-records-in-weft.md) | 32m<br>32m | landing, fabric, report |
| E4 |  | **The Describe text names no LoomYard terms, except the Created using block** | [`describe-no-loomyard-terms`](design-describe-no-loomyard-terms.md) | 32m<br>32m | landing, stencil |
| E5 |  | **Close the scope of a task when its first Code-Review round starts, and have lym hold the line** | [`scope-closes-at-code-review`](design-scope-closes-at-code-review.md) | 32m<br>32m | loom, burler |
| E6 |  | **Review the Describe text inside the Describe row, and catch the rest of what the parent finds at PR-Review earlier** | [`review-describe-text`](design-review-describe-text.md) | 32m<br>32m | landing, burler |
| E7 |  | **Take the parent out of PR-Review; the earlier reviewers must catch what it finds** | [`parent-out-of-pr-review`](design-parent-out-of-pr-review.md) | 32m<br>7m | landing, loom |
| E8 |  | **Autonomy: let PR review stay with the operator for one beam** | [`autonomy-pr-review-operator`](design-autonomy-pr-review-operator.md) | 32m<br>32m | landing, config |

### Features (3)

| # | Prio | Note | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- | --- |
| F1 |  | **A PR mode that registers the PR and approves it at once, with no agent under the block** | [`pr-auto-approve-mode`](design-pr-auto-approve-mode.md) | 32m<br>0m | landing, config |
| F2 |  | **Built-in run watch: mail the parent on every non-running state and on a gate slot held too long** | [`dobby-run-watch`](design-dobby-run-watch.md) | 32m<br>32m | dobby, gate |
| F9 |  | **A closed area for lym: bwrap sandbox in stages, off by default**<br>• Run the lym process tree in a bwrap area with a host proxy and broker, built in three stages with a stop; to be split if it goes ahead | [`sandbox-closed-area-build`](design-sandbox-closed-area-build.md) | 3w<br>5m | reed, shuttle, claude, fabric, creel, dobby, batten, deploy, config, ide |

### Done (1)

| # | Entry | Slug | Age<br>Revised | Labels |
| --- | --- | --- | --- | --- |
| T10 | **Run transitions commit locally; a per-worktree pusher brings them to origin** | [`async-run-pusher`](design-async-run-pusher.md)<br>[Sketch](decision-async-run-pusher.md) | 2d<br>2d | batten, fabric, loom, landing, cli, dobby, process, tender, pattern, heddle |
