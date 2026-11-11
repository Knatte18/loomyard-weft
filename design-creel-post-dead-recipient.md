# creel post to a strand that is not running reports plain success

`creel-post-dead-recipient` · Task · bug, creel · running · Preflight

Imported from [issue #96](https://github.com/Knatte18/loomyard/issues/96).

## Seen

After a machine crash, the hub prime's orch (`dop:orch`) was not running: `lym reed list` showed it `live: false`.
A beam orch posted it a request with `lym creel post dop:orch "<text>"`.
The command printed only `{"id":224,"ok":true,"sender":"..."}`, with no word that the recipient was down.
The sender told the operator the request was on its way, and the operator found the recipient dead only by asking.

## Workaround

Check `lym reed list` (or the session list) for the recipient's `live` flag before relying on a post, and ask the operator to start it.

## Expected

`creel post` to a strand that is not live still queues the message, but says so in its output, for example a `recipient_live: false` field and a warning line naming the strand and how to start it (`lym beam start` for an orch).
