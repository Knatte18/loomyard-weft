# Card 13 — hook-read takes the Stop and filters subagents; hook-wait is the idle waiter

**Create:**
- `plan:internal/creelcli#hookWaitVerb` -> `const hookWaitVerb = "hook-wait"`
- `plan:internal/creelcli#hookWaitCmd` -> `func hookWaitCmd() *cobra.Command`
- `plan:internal/creelcli#hookWait` -> `func hookWait(cmd *cobra.Command, engine string) (exitCode int)`
- `plan:internal/creelcli#runCreelHook` -> `func runCreelHook(cwd, stdin string, stderr io.Writer, args ...string) cliRun`
- `plan:internal/creelcli#hookReadExit` -> `func hookReadExit(event shuttleengine.HookEvent, wrote bool, err error) (exitCode int, printErr bool)`
- `plan:internal/creelcli#TestHookReadExit` -> `func TestHookReadExit(t *testing.T)`

**Edit:**
- `internal/creelcli#hookRead`
- `internal/creelcli#hookReadCmd`
- `internal/creelcli#Command`
- `internal/creelcli#TestCreelCLI`
- `internal/creelcli/doc.go`

**Uses:**
- `internal/shuttleengines#ToolBoundaryRenderer`
- `plan:internal/shuttleengine#HookInput`
- `plan:internal/creelengine#VerbHookWait`
- `plan:internal/creelengine#OutcomeRing`
- `plan:internal/creelengine#BellLine`
- `internal/creelcli#runCreelSplit`
- `internal/creelcli#newRig`
- `plan:internal/testkit/creelfake#HookStrand`
- `plan:internal/shuttleengine#HookEvent`

**Intent:**
D1 to D4 on the CLI side: the hidden verbs a strand's hooks run.

`hook-read <engine>` reads the hook input from its standard input once and parses it with the engine's `HookInput`.
A subagent or fork call (D3) claims nothing and prints nothing, so the mail and its flag stay for the main thread's next boundary; a fork's context dies with it, so mail it took would be lost.
It exits 0 on the tool-boundary event and 1 on a Stop, since on a Stop exit 0 means a block was written and the merged command would then record no turn end; settings hooks fire `SubagentStop` and not `Stop` for a subagent, so this is a guard, but an `agent_id` an in-process teammate carries would otherwise drop a real turn end.
An input that is neither the tool-boundary nor the Stop event is refused on standard error with exit 1, naming the two events the verb takes.
Otherwise it claims with `Request{Verb: VerbClaim, Stop: <event is Stop>, StopHookActive: <the input's flag>}` and writes the claimed lines, joined by newlines, as the engine's `BoundaryBlock` straight to standard output before the client acknowledges them, the order D1 keeps.
On the tool-boundary event it exits 0 whether or not it claimed anything, as before.
On a Stop it exits 0 exactly when it wrote a block, whatever follows the write, and 1 when it claimed nothing, printing nothing to either stream then; a refusal or daemon failure before the write exits 1 with the error on standard error, as today.
`hookRead` reports whether its write callback wrote the block beside its error, and `hookReadExit(event, wrote, err)` decides the exit and whether the error is printed, so the rule is one pure decision: a Stop exits 0 on `wrote` whatever the error, and an error after the write, such as a failed acknowledgement, is still printed to standard error.
That exit status is what card 14's merged Stop command decides on: 0 records no turn end, anything else records it.
The Long text describes the input, the block, the subagent filter and the Stop exit.

`hook-wait <engine>`, hidden like `hook-read`, is D4's waiter: it reads and parses its hook input, exits 0 at once for a subagent call, and otherwise holds `Request{Verb: VerbHookWait, Wait: true}` with no timeout, the hook entry's own timeout bounding it.
On `OutcomeRing` it writes the engine's `RewakeOutput(creelengine.BellLine)` to standard error and exits with its code, 2 for Claude.
A superseded wait, a daemon that cannot be reached or reconnected, and any refusal exit 0 printing nothing, so the waiter never exits 2 but on a ring.
`hookWait` returns the exit code its command sets through `clihelp.SetExit`.
`Command` registers `hookWaitCmd` beside `hookReadCmd`.
Bound: a waiter costs one idle `lym` process per live strand, blocked on its socket (`PATTERN-daemon-wakeups`).

`runCreelHook` runs the creel command as `runCreelSplit` does with stdin as its standard input.
`TestCreelCLI`'s hook rows script their strand as `creelfake.HookStrand` with the test process's pane pid, since a ring reaches only a marked strand, and its existing hook-read row now feeds a PostToolUse input through `runCreelHook`.
It gains rows:
a Stop input with mail of two 5,000-byte bodies, so the budget admits one per claim, prints the block of the first and exits 0, and a second Stop with `stop_hook_active` true, after the first reset the count, claims the second;
a PostToolUse input with mail whose lines, joined, fill the 8 KiB budget exactly and one more message beyond it prints a block whose `reason` and `systemMessage` each equal every line of the budget joined by newlines, whole, and the next call prints the one left;
a Stop with no mail exits 1 printing nothing on either stream;
a PostToolUse input carrying `agent_id` exits 0 printing nothing, and a following main-thread call still claims the mail; a Stop input carrying `agent_id` exits 1 printing nothing and leaves the mail;
an input of another event exits 1 with the refusal on standard error;
a hook-wait as a strand rings when mail is posted: exit 2 and `BellLine` on standard error;
a hook-wait from a subagent input exits 0 at once;
a hook-wait superseded by a newer one of the same strand exits 0 printing nothing;
a hook-wait refused, for an unknown engine, for an input that is not JSON, and from a caller that is no strand (the strand name unset, so the daemon resolves EXTERNAL), each exits 0 printing nothing on either stream.
`TestHookReadExit` is table-driven over the decision: a Stop with a block written and no error, and with a block written and an acknowledgement error, exits 0, the second printing its error; a Stop with nothing written exits 1, printing an error only when there is one; a tool-boundary call exits 0 with nothing written and no error and 1 on an error.
The block-then-failed-acknowledgement case has no CLI row, since nothing outside the daemon can fail an acknowledgement after the write: the day file stays open across an unlink, and the daemon's acknowledgement deadline is 30 seconds.
The creelcli package doc describes both hidden verbs.

**ImpactSummary:** the two hidden hook verbs of creelcli; hook-read's output for a PostToolUse call is unchanged since card 12, and only the hook commands card 14 renders call either verb.
