# Card 5 — Read a fork transcript's pending call from its tail

**Edit:**
- `internal/shuttleengine/claudeengine#holdsPendingToolCall`
- `internal/shuttleengine/claudeengine#transcriptLine`
- `internal/shuttleengine/claudeengine/activity_test.go`
- `internal/shuttleengine/claudeengine/doc.go`

**Uses:**
- `internal/shuttleengine/claudeengine#readBackward`

**Intent:**
D9.
`holdsPendingToolCall` reads the fork transcript backward from its end through `readBackward`, instead of streaming the whole file.
It finds the newest assistant message that holds `tool_use` blocks and reports a pending call when any `tool_use` of that message has no `tool_result` after it.
A message is every assistant entry sharing one `message.id`, since Claude writes each parallel call as its own entry; the read continues back until it has passed every entry of that message, and reads nothing older.
`transcriptLine`'s `Message` gains the `id` field the grouping reads.
A read error, and a transcript with no tool use, report false.
Rationale: an assistant message follows only after every result of the previous one, so older calls are answered, and the cost is bounded by the tail on every turn-end activity read of Merriam.

`TestTurnEndActivity`'s fork rows gain the Fork transcript criterion: a parallel batch whose earlier call is pending and whose last call is answered reads pending;
a fully answered newest message reads not pending whatever older messages hold;
a transcript with no tool use reads not pending.
The existing row "a pending call followed by a sibling's result still reads now" keeps passing.
`claudeengine/doc.go`'s activity paragraph says the pending-call reading looks at the fork's newest tool-calling message only.

**ImpactSummary:** only `forkActivity` calls `holdsPendingToolCall`; an unanswered call in an older message of a fork no longer reads pending, and `transcriptLine`'s other readers ignore the new field.
