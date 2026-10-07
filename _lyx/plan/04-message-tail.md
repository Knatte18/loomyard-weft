# Card 4 — message-tail

**Create:**
- `plan:internal/shuttleengine#MessageTail` -> `const MessageTail = "Answer briefly, then continue your task."`
- `internal/shuttleengine/messagetail.go`
- `internal/shuttleengine/messagetail_test.go`

**Edit:**
- `internal/shuttlecli/send.go`

**Intent:**
D8, the sender half: one constant holds the fixed message tail, and `lyx shuttle send` appends it.

`messagetail.go` declares `MessageTail` and an exported function that returns a text with the tail appended after one space, unless the text, trimmed of trailing whitespace, already ends with `MessageTail`, in which case it returns the text unchanged.
The check skips only the append, never the send: a text that already ends with the tail goes out once with it, never twice.

`lyx shuttle send` passes its text through that function before `Runner.Send`; its help text states that the tail is appended.
`Run.Send` and `Runner.Send` themselves stay unchanged, so the gate's re-prompts and every Go-internal send carry no tail.

`messagetail_test.go` is one table over the function: a plain text gains the tail; a text ending with the tail is unchanged; a text ending with the tail plus trailing spaces or a tab is unchanged; a text containing the tail mid-sentence gains it.

**ImpactSummary:** Every `lyx shuttle send` message now ends with the fixed tail; Go-internal sends are unaffected.
