# lym fabric clone --reset refuses a hub holding leftovers of an earlier _bolt layout

`reset-old-layout-leftovers` · Task · bug, fabric · draft

Depends on: [`fabric-core-push-origin`](design-fabric-core-push-origin.md)

## Seen

The Code-Review Bouncer of `clone-reset-keeps-unpushed` (2026-11-28) reports that a hub holding files of an earlier layout at the `_bolt` root,
such as `.gitrepo-push.lock` and `board.json.swaplock` in the live loomyard hub, still has a plain `lym fabric clone --reset` refused.
No current code writes those files, so the run's rule (lym's own ignored files are no work) does not skip them.

## Idea

Either skip the known leftovers of earlier layouts in the reset's work check, or remove them in a reconcile step, so a plain reset passes on an old hub.
