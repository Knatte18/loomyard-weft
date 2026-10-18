# Card 14 — command-producer

**Create:**
- `plan:internal/shedadapters#CommandRow` -> `type CommandRow struct`
- `plan:internal/shedadapters#CommandDeps` -> `type CommandDeps struct`
- `plan:internal/shedadapters#CommandResult` -> `type CommandResult struct`
- `plan:internal/shedadapters#CommandProducer` -> `type CommandProducer struct`
- `plan:internal/shedadapters#NewCommandProducer` -> `func NewCommandProducer(name string, row CommandRow, deps CommandDeps) *CommandProducer`
- `plan:internal/shedadapters#CommandProducer.Call` -> `func (p *CommandProducer) Call(ctx context.Context) (shedengine.Outcome, shedengine.OutputPointer, error)`
- `plan:internal/shedadapters#expandPlaceholders` -> `func expandPlaceholders(command, goos string, values map[string]string) (expanded, refusedPlaceholder string, err error)`
- `plan:internal/shedadapters#TestCommandProducer` -> `func TestCommandProducer(t *testing.T)`

**Edit:**
- `internal/shedadapters/doc.go`

**Uses:**
- `internal/testkit/logcapture#Capture`
- `plan:internal/verifyrun#QuoteArg`
- `plan:internal/verifyrun#ErrPercentUnderCmd`

**Intent:** Add D4's generic `Command` producer: a row whose switch and command are config settings, which runs the command once through the platform shell and never halts the run over its outcome.

`CommandRow` is what the row's recipe config names: the switch setting's module and key, the command setting's module and key, and the timeout (`timeout_s`, default 30 s, applied by the registry entry in card 15).
`CommandDeps` is what the recipe's owner tells, deriving no path in the producer (PATTERN-told-geometry):
- `Setting`, the plain setting lookup the switch is read through, and through which the producer also reads the command key over the owner's ordinary stack to warn about an ignored value;
- `ExecutableSetting`, the owner's executable-key lookup over the template and the board default layer only, the only lookup the command is read through;
- `Forced`, the row names whose switch the run forces on, read instead of the switch;
- `Lyx`, `Slug` and `TaskDir func() (string, error)`, the placeholder values;
- `Dir`, the directory the command runs in, and `LogPath func(row string) string`, its log file;
- `Run func(ctx context.Context, dir, command, logPath string, timeout time.Duration) (CommandResult, error)`, the runner, `CommandResult` carrying the exit code and whether it timed out;
- `Rerun`, the command that resumes the run (batten: `lyx batten run <slug>`), named in every way forward;
- `GOOS`, empty meaning `runtime.GOOS`, so a test reaches the `cmd` quoting on any platform.

`Call`, in this order:
1. A nil `ExecutableSetting` returns an error before reading or running anything: "shedadapters: <row> (Command): the recipe's owner tells no executable-key lookup; way forward: wire an executable-key lookup over the template and the board default layer in the recipe's owner, then re-run the run on the rebuilt lyx (<Rerun>)".
   The input reaching it: a `CommandDeps` built without it, which `TestCommandProducer` reaches; the registry entry never refuses it at build time, so the refusal falls when the row is reached.
2. A row in `Forced` is on; otherwise the switch is read through `Setting`, and a read error or a non-bool value is an error naming `<module>.<key>` and the way forward "correct it, then <Rerun>".
   A switch that is off is Done with no process, logged at Info.
3. The command is read through `ExecutableSetting`, a read error or a non-string value being the same error for its key.
   When `Setting` resolves the same key to a different non-empty value, the row warns that the value is ignored, naming the way to set it: `lyx config set --layer default` from the prime.
   A `Setting` read of the command key that fails skips that comparison, logged at Info, and never stops the row.
4. An empty command is Done with no process; on a forced row it is logged at Warn.
5. The placeholders `{lyx}`, `{slug}` and `{task_dir}` are replaced, each value quoted by `verifyrun.QuoteArg` for the shell the command runs under; any other brace text, `${HOME}` included, passes through literally, and `TaskDir` is called only when `{task_dir}` appears, its error being the step-2 error naming the task worktree.
   A value `QuoteArg` refuses with `ErrPercentUnderCmd` makes the row Done with no process, warned with the placeholder and its way forward: a path without `%` for the placeholder's source, the hub for `{task_dir}` and the Go bin dir `lyx` is built into for `{lyx}`.
6. The command runs once through `Run` in `Dir` with the log path and timeout; its start and its end are logged at Info (PATTERN-spawn-observability).
   An exit of 0 is Done; a non-zero exit, a timeout and a runner error are each Done, warned with the log path.
   A cancelled context returns the cancellation error, as the switched producer does.
What this lets through: whatever command the template or the board default layer holds runs on the operator's machine with the operator's rights; what bounds it: the command is read only through `ExecutableSetting`, never through a layer a pair can write, it runs once, and `timeout_s` ends a hang.
No command outcome halts the run; only an unreadable setting or a missing executable-key lookup stops it, with the errors above.

`TestCommandProducer`, Tier 1 over fake lookups and a recording `Run`, with `logcapture.Capture` asserting each row's Warn records, table rows:
- a switch off runs nothing and is Done;
- an empty command runs nothing and is Done, with no Warn unforced and a Warn on a forced row;
- exit 0 is Done with one run and no Warn; a non-zero exit, a timeout and a runner error are each Done with one run and a Warn naming the log path;
- an unreadable switch, a switch holding a non-bool, a command lookup error and a non-string command each return an error naming `<module>.<key>` and the `Rerun` text, running nothing;
- a `TaskDir` error with `{task_dir}` in the command returns an error naming the task worktree and the `Rerun` text, running nothing;
- a nil `ExecutableSetting` returns the missing-lookup error naming its way forward, with no lookup read and no run;
- a forced row whose switch lookup would fail runs;
- placeholder values holding spaces, quotes and `%` expand under `linux` to `QuoteArg`'s text;
- under `windows` a `{task_dir}` holding `%` runs nothing, is Done, and warns naming `{task_dir}` and the hub as the way forward;
- `${HOME}` and `{other}` reach the command unchanged;
- a `Setting` answering a different command than `ExecutableSetting` runs the `ExecutableSetting` one and warns that the other is ignored, naming `lyx config set --layer default`; a `Setting` failing on the command key runs the `ExecutableSetting` one with no Warn;
- a cancelled context returns an error.
That a quoted value reaches `sh` as one literal argument is pinned by card 11's `TestRun` row.

`internal/shedadapters/doc.go` gains a `Command` section stating the per-call order, the placeholders, the quoting and the bound; the switched-producer section is removed with that producer in card 15.

**ImpactSummary:** New producer and types only; no registry entry reaches them until card 15, and the doc edit touches no code.

**Commit:** 14: shedadapters: the Command producer
