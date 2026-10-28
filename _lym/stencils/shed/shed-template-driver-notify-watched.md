<!-- This is the shed driver's parent-notification rule for a run something else watches and announces: internal/loomcli's driverPrompt fills it into shed-template-driver as the parent_notify marker, under the `## Notifying the parent` heading.
     It declares no marker and must stay marker-free, and, like the driver stencil, it names no recipe and no recipe-owned command.
lym-stencil: sha256=7e35d73a90cde1d65ef151ba51150d5c83e6eaee2684b9a0760e2babdd2d6141 build=ba2a346b7d63e184b43f4c97ef8a29ae9ae77dab time=2026-10-28T18:20:54Z -->
Message the parent session that the directive above names in exactly three cases, each one short message sent with `lym creel post --parent "<text>"`:

- An `awaiting` stop whose envelope carries a non-empty `parent_notice`: send that notice verbatim, followed by the tail "Answer briefly, then continue your task."
  The notice is the whole message: add nothing to it but that tail, and branch on the envelope field alone.
- A stop report that asks the parent a question you cannot settle yourself: send a message naming the run-id, the stop report's path and the question, ending with that tail, with `lym creel post --parent --expect-reply "<text>"`.
- A `done` stop whose envelope reports `friction: failed`: send a message naming the run-id and the stop report, ending with that tail.

Any other stop, an `awaiting` stop without a `parent_notice` included, sends nothing.
Never message the parent that the run stopped, halted or finished, and send no generic escalation line: those stops reach the parent by other means.
Record every send in the stop report.
When the directive names no parent, or a post fails or is refused, the stop report alone carries the message, with the envelope's `reason`;
record a failed or refused send as a line in the report rather than retrying it.
