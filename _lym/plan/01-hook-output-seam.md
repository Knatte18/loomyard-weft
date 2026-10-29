# Card 1 — hook-output-seam

**Edit:**
- `internal/shuttleengine#ToolBoundaryContextRenderer`
- `internal/shuttleengine#HookInput`
- `internal/shuttleengine/claudeengine/settings.go`
- `internal/shuttleengine/claudeengine#TestHookContext`
- `internal/creelcli#hookReadCmd`
- `internal/creelcli#hookRead`
- `internal/creelcli#hookBlock`
- `internal/creelcli#TestCreelCLI`
- `internal/creelcli#TestHookReadExit`
- `internal/creelengine#Boundary`
- `internal/creelengine#stopBlockCap`
- `internal/shuttleengines#TestToolBoundaryRenderer`
- `internal/shuttleengine/doc.go`
- `internal/shuttleengines/doc.go`
- `internal/shuttleengine/claudeengine/doc.go`
- `internal/creelcli/doc.go`
- `internal/creelengine/doc.go`
- `docs/overview.md`

**Delete:**
- `internal/shuttleengine/claudeengine#Claude.BoundaryBlock`
- `internal/shuttleengine/claudeengine#stopBlockReason`

**Create:**
- `plan:internal/shuttleengine/claudeengine#Claude.HookOutput` -> `func (c *Claude) HookOutput(event shuttleengine.HookEvent, mail, echo string) ([]byte, error)`

**Intent:**
Decisions D1 and D2: neither creel hook delivery is a block any more, so Claude Code adds no command text and no error label.

The seam method `BoundaryBlock(event HookEvent, text string) ([]byte, error)` of `ToolBoundaryContextRenderer` in `internal/shuttleengine/engine.go` is replaced by `HookOutput(event HookEvent, mail, echo string) ([]byte, error)`, an interface method whose change the interface's doc comment states: mail is the claimed lines joined by newlines, echo is D6's operator-only text (the heading line and the echoed lines, which hook-read composes from card 8-hook-read-renders-echo on), and either may be empty.
The interface's doc comment and the doc lines of `internal/shuttleengine/doc.go` and `internal/shuttleengines/doc.go` that say "the visible block" say "the hook output" instead.
`HookInput.StopHookActive`'s field comment says the turn already continues because of an earlier Stop hook's output, not an earlier Stop block.

In `capabilities.go`, `Claude.BoundaryBlock` and the `stopBlockReason` constant are deleted, the file header names the hook output instead of a block, and `Claude.HookOutput` renders, with every empty key omitted (`omitempty`), in this key order:
- after a tool call with mail: `{"systemMessage":<echo + "\n" + mail, or mail alone with no echo>,"hookSpecificOutput":{"hookEventName":"PostToolUse","additionalContext":<mail>}}`;
- on a Stop with mail: `{"systemMessage":<echo>,"hookSpecificOutput":{"hookEventName":"Stop","additionalContext":<mail>}}`, the `systemMessage` key absent with no echo;
- on either event with an echo and no mail: `{"systemMessage":<echo>}`.
No output carries `decision` or `reason`.
The JSON keys stay in claudeengine, so `PATTERN-shuttle-provider-seam` holds and hook-read stays engine-neutral.

`hookRead` in `internal/creelcli/hookread.go` renders the claimed lines with `renderer.HookOutput(input.Event, lines, "")`; its exit codes are unchanged, so a Stop that wrote mail still exits 0 and the merged Stop command records no turn end, while the turn now continues through the Stop hook's additional context (D1's rationale).
`hookReadCmd`'s Short and Long and `internal/creelcli/doc.go` describe the output as the engine's hook output for the event, mail to the model as additional context and, after a tool call, to the operator as the system message, never as a block.
`settings.go` changes comments only: the file header's "a blocked Stop records no turn end" and `creelStopHookCommand`'s doc say that an exit 0 of hook-read means mail was handed to the session, which continues the turn.
`internal/shuttleengine/claudeengine/doc.go` replaces the block paragraph (BoundaryBlock's JSON, the fixed reason and the Bound naming the command text) with D1's and D2's renderings and their rationale, and the Stop-hook paragraph's "block" wording with "mail handed out".

The creel side's wording of a Stop "block" follows: in `internal/creelengine`, the comments of `Boundary` (its `StopHookActive` field) and `stopBlockCap` say a Stop that hands out mail continues the turn and count those continuations, and `internal/creelengine/doc.go`'s claim paragraph says the Stop hook hands out mail as additional context that continues the turn (where it says it "blocks the turn end"), and its post --interrupt paragraph says the Stop claim or the bell delivers the post (where it says "the Stop block delivers it").
`docs/overview.md`'s creel paragraph says a hook strand gets each post at its next tool call in the `PostToolUse` hook's additional context and at its turn end in the `Stop` hook's additional context, which continues the turn and records no turn end, at most 6 a turn.
The names of the test cases that say "block" follow: `TestCreelCLI`'s subtest "as a strand hook-read prints its mail as the engine's visible block, …" names the engine's hook output, and `TestHookReadExit`'s `StopWroteTheBlock…` rows read `StopWroteTheMail…`.

Tests: `TestHookContext` (tier0, `settings_test.go`) keeps its SessionStart row and pins the exact bytes of seven `HookOutput` rows over the text `read "role.md"` as mail and `[creel] Delivered while idle:\nread "role.md"` as echo: tool boundary with mail, with mail and echo, with echo alone; Stop with mail, with mail and echo, with echo alone; each row's want string shows no `decision` and no `reason` key.
`TestToolBoundaryRenderer` (tier0) compares the registry's `HookOutput(HookEventStop, "mail", "")` with the engine's.
`hookBlock(t, event, text)` keeps its signature and returns `HookOutput(event, text, "")` plus a newline, so `TestCreelCLI`'s hook-read subtests (tier1) pin that hook-read prints the new output.

**ImpactSummary:** One interface method replaced and the old implementation deleted; its one production caller (hookRead) and three test sites move with it; comments and docs in five packages and docs/overview.md follow.
