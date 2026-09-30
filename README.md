# Tasks

# Layer A

## **#006:** Landing: board status, PR body, squash message, operator-approved merge, branch cleanup [A]
[landing](design-landing.md)

Landing gaps: #275, #276, #289, #292, #293

## **#007:** Operator surface: what the operator sees and types while a run goes [A]
[operator-surface](design-operator-surface.md)

Operator surface: #295, #277, #286, #297 + four unfiled findings

## **#008:** Burler review rounds: tracked records, focus steering, directive precedence [A]
[burler-review](design-burler-review.md)

Burler: #287, #282, #263 + comment-width finding

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

# Manifest

## **#000:** Fabric: branches left behind by a failed add and on the remote
[fabric-branch-hygiene](design-fabric-branch-hygiene.md)

Parked: #269, #270 — re-check after landing

## **#001:** Batten teardown deletes an llm-driven child's friction notes
[batten-llm-friction](design-batten-llm-friction.md)

Parked: #274 — only matters once batten runs llm-driven children

## **#002:** Kick-start pack: pre-resolved glyph spans for implementer and Burler
[kick-start-pack](design-kick-start-pack.md)

Parked: #227, #228 — measured win, not yet scheduled

## **#003:** Prompt-cache TTL per agent role
[cache-ttl-per-role](design-cache-ttl-per-role.md)

Parked: workers on 5m cache TTL, waiting sessions on 1h — operator deciding
