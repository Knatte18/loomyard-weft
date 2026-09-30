# Tasks

# Layer A

## **#023:** Every refusal has a way forward: audit Webster and shed halt paths [A]
[refusal-audit](design-refusal-audit.md)
Depends on: #022

Walk every refusal/halt path in webster and shed; none may wedge a run

## **#026:** Send PR review findings back into the run instead of fixing outside loom [A]
[publish-review-fixback](design-publish-review-fixback.md)
Depends on: #022

`lyx loom reject <review>` at Publish routes review findings to Webster as a fix batch, then re-verify, re-publish, re-approve.

## **#027:** loom start should refuse a worktree with an unfinished merge [A]
[loom-start-refuses-mid-merge](design-loom-start-refuses-mid-merge.md)

loom start spawned a driver on a worktree whose merge-in had stopped on a conflict (UU path, MERGE_HEAD present).

## **#028:** fabric remove pushes the archive tag before its dirty checks [A]
[archive-tag-before-dirty-check](design-archive-tag-before-dirty-check.md)

A refused teardown (dirty sibling) still pushes archive/<slug>/*; the retry pushes a second tag.

## **#029:** Awaiting waits grow status.json without bound [A]
[awaiting-status-history-growth](design-awaiting-status-history-growth.md)

Each 30 s poll during an unbounded approval wait appends a history entry; days of waiting make a multi-MB status.json.

# Done

## **#000:** Burler: no cluster-exclude warning when the profile has no cluster fan
[burler-exclude-warn](design-burler-exclude-warn.md) [done]

Every Burler round WARNs about dropped cluster excludes; fix at the source (#278)

## **#002:** lyx loom start pane layout: small status band, no loom-operator, agent gets the height
[loom-start-layout](design-loom-start-layout.md) [done]

lyx loom start pane layout (#279)

## **#003:** loom status shows the producer's stuck reason, not 'stuck with no OnStuck target'
[stuck-reason-in-status](design-stuck-reason-in-status.md) [done]

loom status shows the producer's stuck reason, not 'stuck with no OnStuck target'

## **#004:** Plan verify: code-fence lines are not commands
[integration-verify-fence](design-integration-verify-fence.md) [done]

Strip code fences from the plan-level verify command (#285)

## **#006:** Landing: board status, PR body, squash message, operator-approved merge, branch cleanup
[landing](design-landing.md) [done]

Landing gaps: #275, #276, #289, #292, #293

## **#007:** Operator surface: what the operator sees and types while a run goes
[operator-surface](design-operator-surface.md) [done]

Operator surface: #295, #277, #286, #297 + four unfiled findings

## **#008:** Burler review rounds: tracked records, focus steering, directive precedence
[burler-review](design-burler-review.md) [done]

Burler: #287, #282, #263 + comment-width finding

## **#009:** batten opens VS Code on the new pair with reed attached
[batten-open-ide](design-batten-open-ide.md) [done]
Depends on: #020, #021, #019

Batten lifecycle step: open a VS Code window whose terminal is attached to the child run

## **#010:** Batten ready for the hub loop: wait through awaiting, keep run records
[batten-hub-ready](design-batten-hub-ready.md) [done]
Depends on: #007

Batten keeps watching an awaiting child, run records survive teardown, approval time off the watch budget

## **#011:** ide: hub workspace with prime, board and portals
[ide-hub-workspace](design-ide-hub-workspace.md) [done]

One generated VS Code workspace shows the board beside the prime

## **#012:** Publish: verify the tree after merging the parent
[publish-verify-after-merge](design-publish-verify-after-merge.md) [done]

A conflict-resolved parent merge was pushed with non-compiling tests

## **#013:** Webster fork audit: text search for "lyx fabric" counted as a fabric operation
[refscan-false-positive](design-refscan-false-positive.md) [done]
Depends on: #012

RefScanner flags quoted grep patterns; a done batch cannot be recorded

## **#014:** Plan-Write: respawn archives the finished plan; Agent refusal unannounced
[plan-write-respawn](design-plan-write-respawn.md) [done]

Respawn starts over instead of reusing the plan; prompt does not say Agent is refused

## **#015:** Trace logs: configurable retention, fewer files from routine commands
[trace-log-retention](design-trace-log-retention.md) [done]

Trace logs: configurable retention, fewer files from routine commands

## **#016:** lyx quarry: per-glyph resolve answers, grep-safe glyphs output
[quarry-cli-answers](design-quarry-cli-answers.md) [done]

lyx quarry: per-glyph resolve answers, grep-safe glyphs output

## **#017:** Burler: warn when the explore stencil lacks the focus_directive marker
[focus-marker-warn](design-focus-marker-warn.md) [done]

Burler: warn when the explore stencil lacks the focus_directive marker

## **#018:** reed: send a short launch-script line instead of the full command
[reed-launch-script](design-reed-launch-script.md) [done]

Hide the long env+claude launch line behind a generated launch script

## **#019:** One driving surface per run: ly-drive pane or status strand, never both
[status-strand-by-driver](design-status-strand-by-driver.md) [done]
Depends on: #010

llm-driven run shows only ly-drive; go-driven run shows only the status strand

## **#020:** fabric add after remove: leftover remote weft branch rejects the push (flaky test)
[fabric-readd-weft-push](design-fabric-readd-weft-push.md) [done]
Depends on: #010

Re-adding a removed slug races the old remote weft branch; integration test flakes

## **#021:** Webster verify: rerun, compare with parent, halt only on a real regression
[verify-triage](design-verify-triage.md) [done]

Flaky or pre-existing test failures no longer halt a run; report names the failing tests

## **#022:** Webster: a parent merge after a batch commit wedges record-batch
[webster-parent-merge-midrun](design-webster-parent-merge-midrun.md) [done]
Depends on: #021

record-batch demands HEAD == report head_sha; a mid-run merge-in wedges the run

## **#024:** ly-drive: safe self-repairs instead of always handing back
[ly-drive-safe-repairs](design-ly-drive-safe-repairs.md) [done]
Depends on: #010

Driver re-steps known transient stops itself; hands back only real decisions

## **#025:** Hub orchestrator as a reed strand with automatic handoff
[hub-orch-strand](design-hub-orch-strand.md) [done]

lyx hosts the hub orchestrator in tmux and cycles its context via handoff

# Manifest

## **#000:** Fabric: branches left behind by a failed add and on the remote
[fabric-branch-hygiene](design-fabric-branch-hygiene.md)

Parked: #269, #270 — re-check after landing

## **#001:** Batten teardown deletes an llm-driven child's friction notes
[batten-llm-friction](design-batten-llm-friction.md)

Parked: #274 — only matters once batten runs llm-driven children

## **#002:** Kick-start pack: pre-resolved glyph spans for implementer and Burler
[kick-start-pack](design-kick-start-pack.md)

Someday (roadmap): #227, #228 — measured win, committed, not yet scheduled

## **#003:** Prompt-cache TTL per agent role
[cache-ttl-per-role](design-cache-ttl-per-role.md)

Parked: workers on 5m cache TTL, waiting sessions on 1h — operator deciding

## **#008:** Burler: round-N focus directive conflicts with the read-prior-rounds-last rule
[burler-focus-prior-round](design-burler-focus-prior-round.md)

Focus directive needs prior-round files the round prompt says to read only after the review

## **#010:** ide hub workspace: review follow-ups
[ide-hub-workspace-followups](design-ide-hub-workspace-followups.md)

Small cleanups from the PR #301 review

## **#011:** loom start races the ly-drive self-removal and ends with no driver
[driver-restart-race](design-driver-restart-race.md)

First loom start after a hand-back keeps a strand that is about to be removed

## **#012:** Integration tests leave tmux sockets behind
[test-tmux-socket-leak](design-test-tmux-socket-leak.md)

reed/loom tests leak tmux servers and sockets in /tmp

## **#016:** Landing verify gate skips a merge interrupted before its marker write
[verify-gate-crash-after-merge](design-verify-gate-crash-after-merge.md)

A crash between the merge-in commit and the pending-verify marker write leaves the next resume seeing up-to-date with no marker, so verify is skipped.
