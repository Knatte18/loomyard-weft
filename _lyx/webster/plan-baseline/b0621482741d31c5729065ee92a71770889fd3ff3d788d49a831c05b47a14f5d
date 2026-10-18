# Card 3 — creel-command-inject

**Create:**
- `plan:internal/creelengine#CommandName` -> `type CommandName string`
- `plan:internal/creelengine#CommandClear` -> `const CommandClear CommandName = "clear"`
- `plan:internal/creelengine#CommandCompact` -> `const CommandCompact CommandName = "compact"`
- `plan:internal/creelengine#CommandReloadPlugins` -> `const CommandReloadPlugins CommandName = "reload-plugins"`
- `plan:internal/creelengine#CommandColor` -> `const CommandColor CommandName = "color"`
- `plan:internal/creelengine#CommandLoadSkills` -> `const CommandLoadSkills CommandName = "load-skills"`
- `plan:internal/creelengine#Command` -> `type Command struct`
- `plan:internal/creelengine#Command.validate` -> `func (c Command) validate() error`
- `plan:internal/creelengine#Store.InjectCommand` -> `func (s *Store) InjectCommand(sender Sender, receiver string, command Command, ttl time.Duration) (InjectResult, error)`
- `plan:internal/creelcli#sessionCommands` -> `type sessionCommands interface`
- `plan:internal/creelcli#runSessionCommand` -> `func runSessionCommand(ops sessionCommands, guid string, command creelengine.Command) error`
- `plan:internal/creelcli#shuttleOps.RunCommand` -> `func (o shuttleOps) RunCommand(guid string, command creelengine.Command) error`
- `plan:internal/creelcli#hubDelivery.Command` -> `func (d *hubDelivery) Command(strand creelengine.Strand, command creelengine.Command) error`
- `plan:internal/creelcli#stubOps.RunCommand` -> `func (s *stubOps) RunCommand(guid string, command creelengine.Command) error`
- `plan:internal/creelcli#recordingCommands` -> `type recordingCommands struct`
- `plan:internal/creelcli#recordingCommands.ClearSession` -> `func (r *recordingCommands) ClearSession(guid string) error`
- `plan:internal/creelcli#recordingCommands.CompactSession` -> `func (r *recordingCommands) CompactSession(guid, focus string) error`
- `plan:internal/creelcli#recordingCommands.ReloadPlugins` -> `func (r *recordingCommands) ReloadPlugins(guid string) error`
- `plan:internal/creelcli#recordingCommands.TypeColor` -> `func (r *recordingCommands) TypeColor(guid string, color segmentcolor.Color) error`
- `plan:internal/creelcli#recordingCommands.LoadSkills` -> `func (r *recordingCommands) LoadSkills(guid string, skills []string) error`
- `plan:internal/testkit/creelfake#CommandCall` -> `type CommandCall struct`
- `plan:internal/testkit/creelfake#Delivery.Command` -> `func (d *Delivery) Command(strand creelengine.Strand, command creelengine.Command) error`
- `plan:internal/testkit/creelfake#Delivery.Commands` -> `func (d *Delivery) Commands() []CommandCall`
- `plan:internal/testkit/creelfake#Delivery.SetCommandError` -> `func (d *Delivery) SetCommandError(err error)`

**Edit:**
- `internal/creelengine#Delivery`
- `internal/creelengine#Request`
- `internal/creelengine#record`
- `internal/creelengine#injectEntry`
- `internal/creelengine#QueuedInject`
- `internal/creelengine#LogEntry`
- `internal/creelengine#Store.Inject`
- `internal/creelengine#Store.InjectWait`
- `internal/creelengine#Store.inject` -> `func (s *Store) inject(sender Sender, receiver, text string, command Command, ttl time.Duration, interrupt, wait bool) (InjectResult, *Waiter, error)`
- `internal/creelengine#Store.Queued`
- `internal/creelengine#worker.typeInject`
- `internal/creelengine#pool.applyPosted`
- `internal/creelengine#pool.applyCarried`
- `internal/creelengine#pool.carryRecords`
- `internal/creelengine#pool.logEntryFor`
- `internal/creelengine#daemon.dispatch`
- `internal/creelcli#logCmd`
- `internal/creelcli#worktreeOps`
- `internal/creelcli#stubOps`
- `internal/testkit/creelfake#Delivery`
- `internal/creelengine_test#TestStore_InjectAndHold`
- `internal/creelengine_test#TestScheduler_Injects`
- `internal/creelengine_test#TestDaemon_Protocol`
- `internal/creelengine#TestJournal_Fold`
- `internal/creelcli#TestHubDelivery`
- `internal/creelcli#TestSmokeCreelClaudeStrand`
- `internal/creelengine/doc.go`
- `internal/creelcli/doc.go`

**Intent:**
A creel inject carries either a text line, as today, or a command from a closed set (decision record D3).
`internal/creelengine` knows only the neutral `CommandName` and its argument, and no slash spelling or key name (`PATTERN-shuttle-provider-seam`, `PATTERN-creel-store`).

- `Command` holds `Name CommandName` and `Argument string`.
  `validate` refuses a name outside the five constants, a `compact` argument that is empty or holds a control character, a `color` or `load-skills` argument that is empty, and any argument on `clear` or `reload-plugins`; `load-skills` takes a space-separated skill list.
- `Delivery` gains the interface method `Command(strand Strand, command Command) error`, with the same `ErrBusy` and `ErrNotLanded` contract as `Send`.
- `Request` gains `Command` and `Argument` fields; `record`, `injectEntry` and `QueuedInject` carry the command; the fold (`applyPosted`, `applyCarried`) reads it, `carryRecords` carries it, and a record without the fields folds as a text inject (`PATTERN-tolerant-records`).
- `LogEntry` gains `Command` and `Argument`, and `logEntryFor` fills them, plus `Subject` with a text inject's text, on every event of an inject, so a log lookup can tell one step's inject from another's.
  `lyx creel log` then prints a text inject's line as its `subject`; `logCmd`'s help and the `creelengine` doc's `# Log` paragraph say so, and still that no message body is printed.
- `Store.InjectCommand` queues a command inject through `inject`, which `Inject` and `InjectWait` now call with a zero `Command`.
  It refuses a sender that is not a program, a receiver that is not an orch strand, and an invalid command, each naming the rule; it then follows every inject rule: TTL, hold, pause, ordering with text injects, dormant receiver, `sending` before the typing and `injected`, `unconfirmed` or `expired` after, never typed twice.
  Bound: only a program sender, through the Go client, may queue one, and only for an orch strand; the CLI gains no flag, so it can clear, compact or reload only the orch session, only from Go code lyx ships, and only when creel's idle check passes.
- `daemon.dispatch` routes a `VerbInject` request carrying a command to `InjectCommand`, and refuses one that also sets `Wait` or `Interrupt`, naming that a command inject takes neither; input reaching it: a Go client request with a command and `Wait` set.
- `worker.typeInject` types a command inject through `Delivery.Command` and a text inject through `Send`; `ErrBusy` requeues either untouched and `ErrNotLanded` settles either `unconfirmed`.
- `internal/creelcli`: `worktreeOps` gains `RunCommand(guid string, command creelengine.Command) error`; `shuttleOps.RunCommand` calls `runSessionCommand` with its runner, which maps `clear` to `ClearSession`, `compact` to `CompactSession` with the argument as focus, `reload-plugins` to `ReloadPlugins`, `color` to `TypeColor` with `segmentcolor.Color(argument)`, and `load-skills` to `LoadSkills` with the argument's fields, through the `sessionCommands` interface the runner satisfies.
  `hubDelivery.Command` resolves the strand's worktree as `Send` does and maps the error through `mapDeliveryError`, so `shuttleengine.ErrSessionBusy` requeues it.
- `internal/testkit/creelfake`'s `Delivery` records each command call as a `CommandCall` (strand, command), answers `SetCommandError`'s error, and is what the engine tests read.

Tests:
`TestStore_InjectAndHold` gains rows: a program's command inject to `ly:orch` is queued in order behind a text inject; one to `ly:alpha:driver`, one from EXTERNAL, one from `ly:alpha:driver` and one from the other orch strand `ly:orch-2` are refused; each invalid command is refused, as one table: an unknown name, an empty or newline-holding `compact` argument, an empty `color` argument, an empty `load-skills` argument, and an argument on `clear` and on `reload-plugins`; another owner's hold keeps a program's command inject queued.
`TestScheduler_Injects` gains rows: a command inject is typed through `Delivery.Command` and journaled `sending` then `injected`; `ErrNotLanded` journals `unconfirmed`; `ErrBusy` requeues it and the next idle probe types it once; a covering pause holds it and an inject that outlives its TTL expires; a command inject to a dormant orch strand stays queued and is typed once the strand is live again; a text inject, a command inject and a text inject queued to a live orch strand are typed in queue order, which the journal's `sending` events of the three ids show, since `creelfake.Delivery` keeps `Sends()` and `Commands()` as separate lists.
`TestJournal_Fold` gains rows: a posted command inject folds with its command, a record without the fields folds as a text inject, and a carried command inject keeps its command.
`TestDaemon_Protocol` gains rows: a program client's command inject is answered with its id and its log entries carry the command and argument; a command inject with `Wait` set is refused, and one with `Interrupt` set is refused.
`TestHubDelivery` gains rows: each of the five commands reaches the matching `recordingCommands` method with its argument through `runSessionCommand`, and `hubDelivery.Command` maps `ErrSessionBusy` and `ErrSubmissionNotLanded` from `stubOps.RunCommand` onto `ErrBusy` and `ErrNotLanded`.
`TestSmokeCreelClaudeStrand` (llm tier) pins the outcome shuttle reports for each command on a live Claude pane: after its inject step it calls `hubDelivery.Command` for `reload-plugins`, `color`, `load-skills`, `compact` and `clear` on the idle strand, in that order and waiting for idle between them, asserts each answers nil (landed) or `ErrNotLanded` and never `ErrBusy` or another error, and logs which with `t.Log`; the watcher's `unconfirmed` handling (decision record D5) is built for either outcome, so no doc records a run's result.
The llm tier runs only when the operator runs it, since the gate refuses llm-tier runs; the plan's verify vets the file.
The `creelengine` and `creelcli` package docs describe command injects and the new seam operation.

**ImpactSummary:** Adds one operation to creel's Delivery seam, whose two implementations (creelcli's hub delivery and the creelfake kit) both change here, and one Store verb; text injects and every existing caller keep their behavior.
