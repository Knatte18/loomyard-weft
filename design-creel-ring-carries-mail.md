# The idle ring hands the mail over itself, without an empty turn

`creel-ring-carries-mail` · Note · enhancement, creel

## Seen (hub orch, 2026-10-28)

Every piece of mail to an idle hook strand costs an extra model turn.
The `hook-wait` ring wakes the strand with `BellLine`, "Mail is waiting for you. End this turn now", and carries no mail; the woken turn ends with nothing done, and only its Stop hook hands the mail over.
`internal/creelengine/doc.go` states it as the ring's bound: "one ring costs one short model turn before the mail turn".
In the hub orch on 2026-10-28 every notice and agent message arrived this way, so each cost two turns, the first an empty one.

## Idea

The ring hands the mail over itself: when the worker rings, `hook-wait` claims the strand's claimable mail as a boundary claim does, writes it as the rewake output in place of `BellLine`, and acknowledges the claim once the output is written, so the woken turn starts with the mail and no empty turn runs.

## Open

- Whether an asyncRewake output written by a hook that then exits is shown to the model for sure, so acknowledging on write never journals read a message the session did not see; the Stop path's block count guards the same risk there.
- The 8 KiB budget of a claim and whether the rewake output takes the same size.
- A ring that races a turn the operator starts: the mail would land under the operator's prompt, which the Stop path already does today.
