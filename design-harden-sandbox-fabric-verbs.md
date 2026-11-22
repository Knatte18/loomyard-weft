# Hardener rounds may run fabric verbs in their own sandbox hubs

`harden-sandbox-fabric-verbs` · Task · bug, fabric, shed · running · Record-Write

## What was seen (2026-11-22)

Round 1 of `harden-fabricengine` built a one-repository sandbox hub with `lym harden sandbox new --single-repo` and drove it live.
Every mutating `lym fabric` verb was refused from its round agent: add, push, pull, sync, reconcile, checkout, remove, clone and cleanup --apply.
The refusal is `sandboxedRoleRefusal` in `internal/fabriccli/roleguard.go`, which reads only the strand name, so it also refuses verbs run inside the round's own sandbox hub.
Seven of twelve charter items ended as could not verify: the join, the refusals, two clones, the remote delete, pair crash and rebirth, URL variations and every pair verb.

## Idea

Let a hardener round's agent run the mutating fabric verbs in a hub under its own sandbox root, `<worktree>/.lym/harden-sandbox`, and keep the refusal everywhere else.
The guard then checks the hub the verb resolves, not only the strand name, so a verb aimed at the real hub stays refused.
