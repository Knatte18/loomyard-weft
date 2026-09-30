# Tasks

# Layer A

## **#010:** Batten ready for the hub loop: wait through awaiting, keep run records [A]
[batten-hub-ready](design-batten-hub-ready.md)
Depends on: #007

Batten keeps watching an awaiting child, run records survive teardown, approval time off the watch budget

## **#014:** Plan-Write: respawn archives the finished plan; Agent refusal unannounced [A]
[plan-write-respawn](design-plan-write-respawn.md)
Depends on: #013

Respawn starts over instead of reusing the plan; prompt does not say Agent is refused

# Layer B

## **#015:** Trace logs: configurable retention, fewer files from routine commands [B]
[trace-log-retention](design-trace-log-retention.md)
Depends on: #014

Trace logs: configurable retention, fewer files from routine commands

## **#019:** One driving surface per run: ly-drive pane or status strand, never both [B]
[status-strand-by-driver](design-status-strand-by-driver.md)
Depends on: #010

llm-driven run shows only ly-drive; go-driven run shows only the status strand

# Layer C

## **#016:** lyx quarry: per-glyph resolve answers, grep-safe glyphs output [C]
[quarry-cli-answers](design-quarry-cli-answers.md)
Depends on: #015

lyx quarry: per-glyph resolve answers, grep-safe glyphs output

## **#020:** fabric add after remove: leftover remote weft branch rejects the push (flaky test) [C]
[fabric-readd-weft-push](design-fabric-readd-weft-push.md)
Depends on: #019

Re-adding a removed slug races the old remote weft branch; integration test flakes

# Layer D

## **#017:** Burler: warn when the explore stencil lacks the focus_directive marker [D]
[focus-marker-warn](design-focus-marker-warn.md)
Depends on: #016

Burler: warn when the explore stencil lacks the focus_directive marker

## **#021:** Webster verify: rerun, compare with parent, halt only on a real regression [D]
[verify-triage](design-verify-triage.md)
Depends on: #020

Flaky or pre-existing test failures no longer halt a run; report names the failing tests

# Layer E

## **#009:** batten opens VS Code on the new pair with reed attached [E]
[batten-open-ide](design-batten-open-ide.md)
Depends on: #021, #011

Batten lifecycle step: open a VS Code window whose terminal is attached to the child run

## **#018:** reed: send a short launch-script line instead of the full command [E]
[reed-launch-script](design-reed-launch-script.md)
Depends on: #017

Hide the long env+claude launch line behind a generated launch script

# Layer F

## **#012:** Publish: verify the tree after merging the parent [F]
[publish-verify-after-merge](design-publish-verify-after-merge.md)
Depends on: #009

A conflict-resolved parent merge was pushed with non-compiling tests

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

## **#011:** ide: hub workspace with prime, board and portals
[ide-hub-workspace](design-ide-hub-workspace.md) [done]

One generated VS Code workspace shows the board beside the prime

## **#013:** Webster fork audit: text search for "lyx fabric" counted as a fabric operation
[refscan-false-positive](design-refscan-false-positive.md) [done]
Depends on: #012

RefScanner flags quoted grep patterns; a done batch cannot be recorded

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

## **#009:** Webster: a parent merge after a batch commit wedges record-batch
[webster-parent-merge-midrun](design-webster-parent-merge-midrun.md)

record-batch demands HEAD == report head_sha; a mid-run merge-in wedges the run

## **#010:** ide hub workspace: review follow-ups
[ide-hub-workspace-followups](design-ide-hub-workspace-followups.md)

Small cleanups from the PR #301 review
