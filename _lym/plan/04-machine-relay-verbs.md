# Card 4 — machine-relay-verbs

**Create:**
- `plan:internal/creelengine#BridgeProgram` -> `const BridgeProgram = "SLACK"`
- `plan:internal/creelengine#VerbMachineTake` -> `const VerbMachineTake = "machine-take"`
- `plan:internal/creelengine#VerbMachineReport` -> `const VerbMachineReport = "machine-report"`
- `plan:internal/creelengine#VerbMachineSettle` -> `const VerbMachineSettle = "machine-settle"`
- `plan:internal/creelengine#MachineUnsettled` -> `const MachineUnsettled = "bridge-unsettled"`
- `plan:internal/creelengine#MachineOutbound` -> `type MachineOutbound struct`
- `plan:internal/creelengine#IsDaemonRefusal` -> `func IsDaemonRefusal(err error) bool`
- `plan:internal/creelengine#ErrDaemonRefusal` -> `var ErrDaemonRefusal = errors.New("creel: the daemon refused the request")`
- `plan:internal/creelengine#daemonRefusal.Is` -> `func (e *daemonRefusal) Is(target error) bool`
- `plan:internal/creelengine#Store.requireBridgeProgram` -> `func (s *Store) requireBridgeProgram(sender Sender, verb string) error`
- `plan:internal/creelengine#Store.TakeMachineOutbound` -> `func (s *Store) TakeMachineOutbound(sender Sender) ([]MachineOutbound, error)`
- `plan:internal/creelengine#Store.ReportMachineHold` -> `func (s *Store) ReportMachineHold(sender Sender, id int64, reason string) error`
- `plan:internal/creelengine#Store.SettleMachineDelivered` -> `func (s *Store) SettleMachineDelivered(sender Sender, id int64, slackTS string) error`
- `plan:internal/creelengine#Store.SettleMachineReturned` -> `func (s *Store) SettleMachineReturned(sender Sender, id int64, reason string) error`
- `plan:internal/creelengine#Store.returnMachineLocked` -> `func (s *Store) returnMachineLocked(entry *machineOutboundEntry, reason string, listing strandListing) error`
- `plan:internal/creelengine#Store.returnDueMachine` -> `func (s *Store) returnDueMachine() (time.Time, bool, error)`
- `plan:internal/creelengine#Store.SetMachineHook` -> `func (s *Store) SetMachineHook(hook func())`
- `plan:internal/creelengine#Scheduler.machineHourLoop` -> `func (s *Scheduler) machineHourLoop()`
- `plan:internal/creelengine#Scheduler.wakeMachineHour` -> `func (s *Scheduler) wakeMachineHour()`
- `plan:internal/creelengine_test#TestStore_MachineOutbound` -> `func TestStore_MachineOutbound(t *testing.T)`

**Edit:**
- `internal/creelengine#Request`
- `internal/creelengine#Response`
- `internal/creelengine#daemon.dispatch`
- `internal/creelengine#Store`
- `internal/creelengine#Store.do`
- `plan:internal/creelengine#Store.postMachine`
- `internal/creelengine#Scheduler`
- `internal/creelengine#NewScheduler`
- `internal/creelengine#Scheduler.Start`
- `internal/creelengine#Store.noticeReturnedLocked` -> `func (s *Store) noticeReturnedLocked(id int64, from, to, reason string, listing strandListing) error`
- `internal/creelengine#Store.settleReturned`
- `internal/slackengine#SlackProgram`
- `cmd/lym#unboundedWaitFiles`
- `cmd/lym#testOnlyCodeAllowed`

**Extends:**
- `internal/creelengine/doc.go`
- `pattern/PATTERN-daemon-wakeups.md`

**Uses:**
- `internal/creelengine#outboundDeadline`
- `internal/creelengine#daemonRefusal`
- `internal/creelengine#Store.listStrands`
- `internal/creelengine#Store.chaserSender`
- `internal/creelengine#Store.appendPostedLocked`
- `internal/creelengine#Store.deliverMailLocked`
- `internal/creelengine#Store.answerLocked`
- `plan:internal/creelengine#eventMachineHeld`
- `plan:internal/creelengine#eventMachineDelivered`
- `plan:internal/creelengine#eventMachineReturned`
- `internal/testkit/creelfake#NewClock`
- `plan:internal/creelengine_test#env.withBridge`

**Intent:**
D3's relay verbs and creel's hour: the program-only `take`, `report` and `settle`, and the scheduler timer that returns an unsettled machine-outbound message an hour after its post.
The new code goes in a new file `internal/creelengine/machinemail.go`, which `cmd/lym#unboundedWaitFiles` lists, since it holds the hour loop's unbounded wait.

`BridgeProgram` is the relay's program name, `SLACK`; `slackengine.SlackProgram` becomes `= creelengine.BridgeProgram`, so one constant names it (see `## Shared Decisions`).
`requireBridgeProgram` accepts only a program sender named exactly `<shortname>:SLACK` of this hub (a scoped `<shortname>:<slug>:SLACK` is refused) and refuses any other with `creel: only %s may %s bridge mail, and %s is not it; way forward: post to another machine with lym creel post <machine>::<name> "<text>"`.
Bound: on Linux a program claim through the Go client is accepted from an orch strand's pane and from a process in no pane, and off Linux from any process; either can make the `SLACK` claim and take, settle or deliver bridge mail, as creel's existing program-claim bound already allows for this program.

`MachineOutbound` is what take hands the relay: `ID`, `From`, `To`, `ReceivingMachine`, `Body`, `ReplyTo`, `ExpectReply` and `Posted`.
`TakeMachineOutbound` first returns every pending message whose `Posted` plus `outboundDeadline` is at or before now (through `returnMachineLocked`, reason its hold reason or `MachineUnsettled`), then answers the pending ones oldest first; a message past its hour is therefore never handed out.
`ReportMachineHold` journals `machine-held` with the reason only when it differs from the entry's hold reason, as `stalled` is written; a report for a settled message writes nothing; an id creel holds no machine-outbound message for is refused naming the id, with the way forward `take again; creel no longer holds that message`.
`SettleMachineDelivered` journals `machine-delivered` with the Slack timestamp for a pending message; an id creel holds no machine-outbound message for, such as one retention pruned, is refused naming the id, which the relay takes as answered (card 10).
For a message creel returned, it journals `machine-delivered` and posts the sender a `CREEL` notice: `Message #%d to %s, which came back at its hour, did reach the other machine; see: lym creel log --id %d`.
For a message already delivered it writes nothing and posts nothing.
`SettleMachineReturned` returns a pending message with the relay's reason code, and leaves a settled one alone.
`returnMachineLocked` journals `machine-returned`, answers a reply wait as returned with `message()`, and posts the return notice through `noticeReturnedLocked`.
`noticeReturnedLocked` is re-signed to take the id, sender, receiver and reason, so both the per-hub and the machine return word one notice; `settleReturned` passes its entry's fields.
For a machine return the notice's way forward also names what clears the hold reason, since posting again only holds the message again until the operator acts: `peer-unapproved` asks the operator to type `approve <machine>` in an orch channel of this machine; `bridge-name-taken` and `bridge-archived` point at the relay notice in the orch's channel; `button-off` names the slack button; any other code points at the relay notice for that code.
`IsDaemonRefusal` reports whether err is or wraps a refusal the daemon answered, so the relay tells an answered settle from a lost connection (card 10).
It is `errors.Is(err, ErrDaemonRefusal)`, and `daemonRefusal.Is` answers true for `ErrDaemonRefusal`, so every refusal `Client.do` builds matches it.
A fake outside the package stands for an answered refusal by returning an error that wraps `ErrDaemonRefusal`.
No main reaches `IsDaemonRefusal` until card 10's relay settles through it, so `cmd/lym#testOnlyCodeAllowed` gains `internal/creelengine.IsDaemonRefusal` with the reason `no caller until card 10's relay tells an answered settle from a lost connection; that card removes this entry`.

`Request` gains `Reason string`, `Delivered bool` and `SlackTS string`; `Response` gains `MachineOutbound []MachineOutbound`.
`daemon.dispatch` serves `machine-take` (answering the list), `machine-report` (`ID`, `Reason`) and `machine-settle` (`ID` with `Delivered` and `SlackTS`, or `Reason`), each through the resolved sender, so `requireBridgeProgram` judges it.

The hour timer: `Store` gains `machineHook func()` and a pending-wake flag; `postMachine` sets the flag when it journals, and `Store.do` calls the hook after the mutex is free, as it calls the outbound hook.
`Scheduler` gains a `machineWake` channel of one slot, made by `NewScheduler`; `Start` sets the hook to `wakeMachineHour`, which fills the slot, and starts `machineHourLoop`.
`machineHourLoop` calls `returnDueMachine`, which returns every due message and answers the earliest hour still pending; it then waits until that hour on the scheduler's clock, never less than a second, or with none pending until a wake; a wake or stop ends the wait.
So the timer runs whether or not the relay runs, the start scan arms it through its first pass, and a post arms it through the hook.

`TestStore_MachineOutbound` (tier 0) is table-driven, each row `{name, run func(t, *env)}` over `newEnv(t).withBridge(...)` and the env's `creelfake.Clock`:
- take answers two pending messages oldest first with their reply-to;
- an unchanged hold reason reported three times writes one `machine-held` record and a changed one writes a second; a report for a settled message writes nothing;
- a message settled delivered is not returned at its hour, and a settle returned for it writes nothing;
- with the scheduler started and no take, a message is returned at its hour with its last reported reason, and one with none with `bridge-unsettled`, each with one `CREEL` notice to the sender naming the reason's way forward (`approve <machine>` for `peer-unapproved`) and a reply wait answered returned;
- two messages posted half a second apart are returned in one pass, and the hour loop's recorded waits on the clock are never under a second;
- a take after the hour returns the message and never answers it;
- a settle delivered after the return journals `machine-delivered` and posts the late notice, and a second writes nothing and posts nothing;
- a settle returned with `channel_not_found` returns the message with that reason;
- take, report and settle from an orch strand, another program, a scoped `SLACK` and EXTERNAL are each refused naming `lym creel post`;
- a report and a settle delivered naming an unknown id are each refused naming it.
`internal/creelengine/doc.go`'s machine section gains the verbs, the program check, its bound and the hour; `pattern/PATTERN-daemon-wakeups.md`'s list of creel's loops gains the hour loop, waking at the earliest pending hour and never within a second.

**ImpactSummary:** New verbs and one scheduler goroutine in `internal/creelengine`; the re-signed `noticeReturnedLocked` has one caller, `settleReturned`, changed here; `slackengine.SlackProgram` keeps its value.

**Commit:** 4: creel's relay verbs and the machine mail hour
