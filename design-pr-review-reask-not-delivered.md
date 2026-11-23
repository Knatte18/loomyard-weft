# PR-Review re-ask is never delivered: pr-reviewer strand may not sign as the program LOOM

`pr-review-reask-not-delivered` · Note · bug, creel, loom

Imported from [issue #200](https://github.com/Knatte18/loomyard/issues/200).

Import note: the issue names files under `.lym/`, which is deleted with its worktree, so only what the issue quotes of them reached the board: `.lym/parent-review/asks.json`.

Seen in hub mo, run hydraulics-gaps, at PR-Review with pr_review_ask_parent on. Ask 3, sent by the row, reached the parent orch and was answered with findings. Asks 2 and 4, sent from the PR-Review agent's own session, were recorded in .lym/parent-review/asks.json as state no-parent, with reason: 'creel: mo:hydraulics-gaps:pr-reviewer may not sign as the program mo:LOOM: only an orch strand's pane may claim a program name'. The parent orch was live the whole time. The agent read no-parent as the parent being down and ended its turn, so the review stalled. Workaround: the parent mails the PR-Review agent directly and points it at the earlier ask's report. Expected: a re-ask from the PR-Review session is delivered (signed by a sender allowed to claim it, or sent through the row), and a refused delivery is recorded as a delivery failure, not as no-parent.