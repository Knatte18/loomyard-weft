# Card 8 — hook-read-renders-echo

**Edit:**
- `internal/creelcli#hookRead`
- `internal/creelcli#hookReadCmd`
- `internal/creelcli#hookReadExit`
- `internal/creelcli#TestCreelCLI`
- `internal/shuttleengine/claudeengine#creelStopHookCommand`
- `internal/shuttleengine/claudeengine#TestCreelStopHook_RecordsATurnEndOnlyWithoutABlock`
- `internal/creelcli/doc.go`
- `internal/shuttleengine/claudeengine/doc.go`

**Uses:**
- `plan:internal/creelengine#EchoHeading`
- `internal/creelcli#newHookRig`
- `internal/creelcli#runCreelHook`
- `internal/creelcli#hookLine`

**Intent:**
Decision D6, hook side: `hook-read` shows the operator the echo as the system message under `[creel] Delivered while idle:`, and a Stop that hands out an echo and no mail lets the turn end.

`hookRead`'s handler renders `renderer.HookOutput(input.Event, mail, echo)`, where mail is the response's message lines joined by newlines and echo is `creelengine.EchoHeading`, a newline and the response's `Echo` lines joined by newlines, empty when `Echo` is empty; it runs whenever the daemon asks an acknowledgement, so an echo-only claim is written too.
`hookRead`'s `wrote` result now reports that mail was written; an echo alone leaves it false.
A failed write of the output says what stays: "could not write the hook output, so its mail stays unread and its echo stays owed", the mail half or the echo half alone when the claim handed out only one, since an echo is never unread mail.
`hookReadExit` keeps its rule over `wrote`, so a Stop exits 0 exactly when mail was written and 1 otherwise, an echo-only output included; its doc comment says so, and that the merged Stop command keeps an exit-1 output as the hook's while it records the turn end.
`hookReadCmd`'s Long and `internal/creelcli/doc.go` state the echo's rendering and the exit rule.
`creelStopHookCommand`'s doc comment and `internal/shuttleengine/claudeengine/doc.go`'s merged-command paragraph say that an exit other than 0 records the turn end with whatever hook-read printed left as the hook's output, which D6's echo relies on.

Tests:
- (tier1) `TestCreelCLI` new subtest `after a ring hook-read shows the echo to the operator alone, ahead of any mail`: a post, `hook-wait` exits 2; a Stop `hook-read` prints `HookOutput(stop, "", EchoHeading+"\n"+hookLine(1, alpha, "mail"))` and a newline and exits 1; a second Stop prints nothing and exits 1; a second post and `hook-wait` ring, a third post, and a tool-boundary `hook-read` prints `HookOutput(tool-boundary, hookLine(3, …), EchoHeading+"\n"+hookLine(2, …))` and exits 0.
- `TestCreelStopHook_RecordsATurnEndOnlyWithoutABlock` (tier1, as its file): the `block` constant becomes the mail output `{"hookSpecificOutput":{"hookEventName":"Stop","additionalContext":"mail"}}`, and a new row `FlagStandsAndAnEchoAloneIsWritten` (stub exit 1 printing `{"systemMessage":"echo"}`) wants that line on stdout and the turn end recorded.

**ImpactSummary:** hookRead and hookReadExit keep their signatures; only the rendered output and the echo-only Stop exit change, with the merged command's shell text untouched.
