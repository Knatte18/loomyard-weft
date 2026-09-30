# Tasks

# Layer A

## **#010:** Batten ready for the hub loop: wait through awaiting, keep run records [A]
[batten-hub-ready](design-batten-hub-ready.md)
Depends on: #007

Batten keeps watching an awaiting child, run records survive teardown, approval time off the watch budget

# Layer B

## **#009:** batten opens VS Code on the new pair with reed attached [B]
[batten-open-ide](design-batten-open-ide.md)
Depends on: #010

Batten lifecycle step: open a VS Code window whose terminal is attached to the child run

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

## **#004:** Trace logs: configurable retention, fewer files from routine commands
[trace-log-retention](design-trace-log-retention.md)

Trace logs: configurable retention, fewer files from routine commands

## **#005:** lyx quarry: per-glyph resolve answers, grep-safe glyphs output
[quarry-cli-answers](design-quarry-cli-answers.md)

lyx quarry: per-glyph resolve answers, grep-safe glyphs output

## **#006:** Burler: warn when the explore stencil lacks the focus_directive marker
[focus-marker-warn](design-focus-marker-warn.md)

Burler: warn when the explore stencil lacks the focus_directive marker

## **#007:** Webster fork audit: text search for "lyx fabric" counted as a fabric operation
[refscan-false-positive](design-refscan-false-positive.md)

RefScanner flags quoted grep patterns; a done batch cannot be recorded

## **#008:** Burler: round-N focus directive conflicts with the read-prior-rounds-last rule
[burler-focus-prior-round](design-burler-focus-prior-round.md)

Focus directive needs prior-round files the round prompt says to read only after the review

## **#009:** Publish: verify the tree after merging the parent
[publish-verify-after-merge](design-publish-verify-after-merge.md)

A conflict-resolved parent merge was pushed with non-compiling tests

## **#010:** Plan-Write: respawn archives the finished plan; Agent refusal unannounced
[plan-write-respawn](design-plan-write-respawn.md)

Respawn starts over instead of reusing the plan; prompt does not say Agent is refused

## **#011:** reed: send a short launch-script line instead of the full command
[reed-launch-script](design-reed-launch-script.md)

Hide the long env+claude launch line behind a generated launch script

## **#012:** ide: hub workspace with prime, board and portals
[ide-hub-workspace](design-ide-hub-workspace.md)

One generated VS Code workspace shows the board beside the prime
