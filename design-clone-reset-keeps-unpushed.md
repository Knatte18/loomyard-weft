# lym fabric clone --reset removes unpushed records, board commits and uncommitted Bolt changes

`clone-reset-keeps-unpushed` · Task · bug, fabric · running · Darn-Write

## Seen

The landing of `hardener-study-faults` (`7cb004818`) says `lym fabric clone --reset` still removes unpushed records and board commits,
uncommitted Bolt changes and ignored files, and leaves open whether that is acceptable.

## Operator decision (2026-11-28)

It is not acceptable: `--reset` refuses when it would lose work, and names what would be lost.

## Idea

Before `--reset` removes anything, check for unpushed records and board commits, uncommitted Bolt changes and ignored files that hold work.
When any is found, refuse with the list and the way forward: push or commit them first, or pass an explicit flag that accepts the loss.
