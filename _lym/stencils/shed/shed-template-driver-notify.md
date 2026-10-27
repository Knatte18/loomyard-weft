<!-- This is the shed driver's parent-notification rule for a run nothing else watches: internal/loomcli's driverPrompt fills it into shed-template-driver as the parent_notify marker, under the `## Notifying the parent` heading.
     It declares no marker and must stay marker-free, and, like the driver stencil, it names no recipe and no recipe-owned command.
lyx-stencil: sha256=8bf600fc47bdf6e0b63b305e5a232dccfd59c37fa9ea25a36b188147452df12a
lym-stencil: sha256=8bf600fc47bdf6e0b63b305e5a232dccfd59c37fa9ea25a36b188147452df12a build=ef912d4155479098331ad98253e9a38250450e09 time=2026-10-27T07:47:27Z -->
At every escalation, after writing the stop report, send the parent session that the directive above names one short SendMessage naming the run-id and the stop report's path and nothing more than the tail "Answer briefly, then continue your task."
At an `awaiting` stop whose envelope carries a non-empty `parent_notice`, send that notice verbatim, followed by that tail, instead of this generic line, and record the send in the stop report.
The notice is the whole message: add nothing to it but that tail, and branch on the envelope field alone.
When the directive names no parent, or the send fails, the stop report alone is the escalation, with the envelope's `reason`;
record a failed send as a line in the report rather than retrying it.
A stop at `done` sends nothing, since nothing awaits a decision, except a `done` stop whose envelope reports `friction: failed`: it notifies the parent as an escalation does, naming the run-id and the stop report.
