# Card 12 — The tool-boundary capability parses hook input and renders a visible block per event

**Create:**
- `plan:internal/shuttleengine#HookEvent` -> `type HookEvent string`
- `plan:internal/shuttleengine#HookEventToolBoundary` -> `const HookEventToolBoundary HookEvent = "tool-boundary"`
- `plan:internal/shuttleengine#HookEventStop` -> `const HookEventStop HookEvent = "stop"`
- `plan:internal/shuttleengine#HookInput` -> `type HookInput struct`
- `plan:internal/shuttleengine/claudeengine#Claude.HookInput` -> `func (c *Claude) HookInput(payload []byte) (shuttleengine.HookInput, error)`
- `plan:internal/shuttleengine/claudeengine#Claude.BoundaryBlock` -> `func (c *Claude) BoundaryBlock(text string) ([]byte, error)`
- `plan:internal/shuttleengine/claudeengine#Claude.RewakeOutput` -> `func (c *Claude) RewakeOutput(text string) (stderr []byte, exitCode int)`
- `plan:internal/shuttleengine/claudeengine#TestClaude_HookInput` -> `func TestClaude_HookInput(t *testing.T)`

**Edit:**
- `internal/shuttleengine#ToolBoundaryContextRenderer`
- `internal/creelcli#hookRead`
- `internal/shuttleengine/claudeengine#TestHookContext`
- `internal/shuttleengines#TestToolBoundaryRenderer`
- `internal/creelcli#TestCreelCLI`
- `internal/shuttleengine/claudeengine/doc.go`
- `internal/shuttleengines/doc.go`
- `internal/shuttleengine/doc.go`

**Delete:**
- `internal/shuttleengine/claudeengine#Claude.ToolBoundaryContext`

**Uses:**
- `internal/shuttleengines#ToolBoundaryRenderer`
- `internal/shuttleengine/claudeengine#hookEventStop`
- `internal/shuttleengine/claudeengine#hookEventPostToolUse`

**Intent:**
D1 and D3: hook input parsing and hook output stay under `claudeengine`, behind `shuttleengine.ToolBoundaryContextRenderer`, which now renders per event and answers the subagent check; `creelcli` and `creelengine` see only the capability (`PATTERN-shuttle-provider-seam`).

`HookInput` is what a creel hook needs from the provider's hook input: `Event HookEvent`, `HookEventToolBoundary` for the hook after a tool call, `HookEventStop` for the turn-end hook and empty for any other event; `Subagent bool`, set when the call comes from a subagent or fork; `StopHookActive bool`, the turn-end input's flag that the session already continues because of a Stop block.
`ToolBoundaryContextRenderer` loses `ToolBoundaryContext` and gains three methods, its doc saying the capability renders creel's hook delivery:
`HookInput(payload []byte) (HookInput, error)`, the parse;
`BoundaryBlock(text string) ([]byte, error)`, the output that hands text to the session and shows it to the operator, the same for both events;
`RewakeOutput(text string) (stderr []byte, exitCode int)`, what an idle waiter hook writes and exits with to wake the session.
Claude implements them: `HookInput` reads `hook_event_name` (`PostToolUse` and `Stop` map to the two events), a non-empty `agent_id` as `Subagent` (never `agent_type`, which `--agent` also sets), and `stop_hook_active`, refusing a payload that is not a JSON object with an error naming the parse;
`BoundaryBlock` marshals `{"decision":"block","reason":<text>,"systemMessage":<text>}`, so the model reads the reason and the operator sees the system message under the tool result or the stop;
`RewakeOutput` returns text plus a newline and exit code 2, the `asyncRewake` wake.
`Claude.ToolBoundaryContext` goes; `hookContext` stays for `SessionStartContext`.
`capabilities.go`'s file header names creel's hook delivery, the hook-input parse, the visible block and the wake output, in place of tool-boundary context rendering.

Its callers move in this card: `hookRead` renders its claimed lines with `BoundaryBlock` in place of `ToolBoundaryContext`, so the PostToolUse hook already prints the visible block; card 13 adds the input, the Stop and the subagent filter.
`TestHookContext` keeps its SessionStart row and replaces its tool-boundary row with a `BoundaryBlock` row pinning the block JSON byte for byte, `"` in the text escaped once in each field.
`TestClaude_HookInput` is table-driven over payloads: a main-thread `PostToolUse`, a `Stop` with `stop_hook_active` true and false, a `PostToolUse` carrying `agent_id` (Subagent true), one carrying only `agent_type` (Subagent false), an unrelated event (empty event) and a payload that is not JSON (an error); one row pins `RewakeOutput`'s text and exit code 2.
`TestToolBoundaryRenderer` compares the registry's `BoundaryBlock` with the engine's.
`TestCreelCLI`'s hook-read row wants the block JSON of the two quoted lines.
The claudeengine doc's creel paragraph names the capability's three methods and the block shape.
The package docs outside claudeengine that call it a context renderer follow: the shuttleengines package doc's line on it and the shuttleengine package doc's list of optional capabilities each describe creel's hook delivery, the input parse, the visible block and the wake output.
`shuttleengines.ToolBoundaryRenderer`'s own doc comment was rewritten in card 8 in terms that stay true here.

**ImpactSummary:** the optional capability's method set changes; its one implementer is Claude, its one production caller creelcli's hook-read, and its test callers sit in three packages, all in this card.
