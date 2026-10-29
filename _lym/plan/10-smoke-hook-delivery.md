# Card 10 — smoke-hook-delivery

**Edit:**
- `internal/creelcli#TestSmokeCreelHookDelivery`

**Create:**
- `internal/creelcli#claudeScenario.transcriptCount`

**Uses:**
- `internal/creelcli#forkTranscripts`
- `internal/creelcli#smokeHub.lastPane`
- `internal/creelcli/doc.go`

**Intent:**
The real-session acceptance criteria of D1, D2, D3 and D6 against Claude Code, in the tier3 smoke test `TestSmokeCreelHookDelivery` (`smoke_llm_test.go`, `//go:build tier3`), which the operator runs by hand with a `lym` built from this tree on PATH; no gate runs tier3, and the gates' vet step compiles it.

`claudeScenario.transcriptCount` counts the occurrences of text in the session's main transcript, found as `forkTranscripts` finds it, from the first `transcript_path` the events hold.

Subtests:
- `mid-turn post is delivered at the next tool call` also requires the transcript to hold no `lym creel hook-read` command text and no `hook blocking error` (D2).
- `a post after the last tool call is delivered by the Stop block and the turn goes on` is renamed `… delivered at the Stop as additional context and the turn goes on`, keeps its one-Stop-payload and one-stamp checks, and requires no hook command text and no `hook blocking error` in the transcript (D1).
- A new subtest `a Stop claim of more than five lines shows the operator its head once`: after the last tool call seven short posts land in one Stop claim; the model acts on the seventh, and in the pane's stop-hook summary the first post's token appears once in its Stop hook feedback lines and no second `Stop hook feedback:` row and no `Stop says:` line carries any post's token; the model's own reply and tool calls may quote a token.
- `an idle session with a draft is woken and keeps its draft` and `successive wakes are each delivered through the Stop block` (renamed `… through the waiter`) also require each wake's token in the transcript within the first message of the woken turn, so no turn without the mail ran before it, the pane to show one `creel mail` row per wake, and the pane to show `[creel] Delivered while idle:` with the token once per wake, the echo, by the woken turn's end (D3, D6).
- `post --interrupt stops the first turn and the bell wakes the session once` stays as is, since the bell's path is out of scope.

**ImpactSummary:** One tier3 test and one new helper beside it; no production code and no automatic gate affected.
