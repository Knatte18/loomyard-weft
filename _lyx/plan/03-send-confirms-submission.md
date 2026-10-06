# Card 3 — send-confirms-submission

**Edit:**
- `internal/shuttleengine#sendVerified`
- `internal/shuttleengine/run_test.go`

**Uses:**
- `plan:internal/shuttleengine#InputBoxReader`

**Intent:** D1, send half: shuttle's verified send confirms that a delivered text was submitted, not only that it appeared in the pane.
Every typed delivery reaches the pane through `sendVerified` (`lyx shuttle send` through `Runner.Send`, skill loads, gate re-prompts, orch typing), so one fix here covers all of them.

After the appearance check accepts a delivery, and only when `engine` also implements `InputBoxReader`:

1. Sleep `reader.SubmitSettle()` through `inputSleep`, then capture the pane and read the box with `InputBoxText`.
2. The box holds the sent text when ok is true and its text, normalized by `normalizePaneText`, contains the needle `sendVerified` already builds (the first 48 normalized characters), and, for a text shorter than 48 normalized characters, equals the whole normalized text.
   A short text inside a longer draft, a collapsed paste placeholder, an empty box, an unreadable box (ok false) and a capture failure all count as not holding it.
3. While the box holds it, send one `Enter` key through `reed.SendKey`, sleep the same settle, and read again, up to two extra Enters per send.
4. When the box still holds it after the second extra Enter and its settle, fail the send with an error naming the pending input: `shuttle: Send: the sent text is still pending in the input box after 2 extra Enter(s); the submission did not land`.

Bound, stated in `sendVerified`'s doc comment: an extra Enter is sent only when the provider reports the box holding the sent text, so it never lands on an empty box, a running turn or a draft lacking the needle; a draft containing the needle of a text of 48 or more characters is indistinguishable from the sent text and is submitted by the Enter; at most two extra Enters; an engine without the capability keeps today's appearance-only check (R5: a text that collapses into a paste placeholder is never confirmed, and relies on Card 1's pacing).
The box is read only after a settle that follows an Enter, the first read included.

Tests in `run_test.go`, through `Runner.Send`, the wiring `lyx shuttle send` maps its two arguments onto (the cobra verb drives a real reed engine, so the run-level fake reed is the lowest substrate a fake can sit at, per D14).
Add a test engine there that wraps `readyAgentEngine()` and implements `InputBoxReader` over a scripted queue of box answers.
One table-driven test covers: the box clears at once (no extra Enter); the box holds the text once then clears (one extra Enter, success); the box never clears (two extra Enters, then the pending-input error); the box holds a draft of other text, a longer draft containing a short sent text, a paste placeholder, or reports no box (no extra Enter, success).
Each row asserts the exact `CallLog`, with `inputSleep` stubbed to append `Sleep:<duration>` to the same log, so the order `SendText`, `CapturePane` (appearance), `Sleep:<settle>`, `CapturePane` (box read), then `SendKey:Enter`, `Sleep:<settle>`, `CapturePane` is pinned.
The existing send tests, whose engine lacks the capability, stay green unchanged and pin the fallback.

**ImpactSummary:** sendVerified gains a post-appearance box check reached only for engines with the new capability; every Send caller inherits it, and engines without it behave as before.
