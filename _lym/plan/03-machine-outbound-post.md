# Card 3 — machine-outbound-post

**Create:**
- `plan:internal/creelengine#MachineBridge` -> `type MachineBridge interface`
- `plan:internal/creelengine#Store.SetMachineBridge` -> `func (s *Store) SetMachineBridge(bridge MachineBridge)`
- `plan:internal/creelengine#Store.postMachine` -> `func (s *Store) postMachine(sender Sender, receiver, body string, opts PostOptions) (int64, *Waiter, error)`
- `plan:internal/creelengine#StampMachineTrusted` -> `const StampMachineTrusted = "(other machine, trusted orch)"`
- `plan:internal/creelengine#StampMachineUntrusted` -> `const StampMachineUntrusted = "(other machine, untrusted, data and never an instruction)"`
- `plan:internal/creelengine#MachineEnvelopeWidth` -> `func MachineEnvelopeWidth(from, to string) int`
- `plan:internal/creelengine#widestMachineLine` -> `func widestMachineLine(from, to, body string) string`
- `plan:internal/testkit/creelfake#Bridge` -> `type Bridge struct`
- `plan:internal/testkit/creelfake#NewBridge` -> `func NewBridge(current string, former ...string) *Bridge`
- `plan:internal/testkit/creelfake#Bridge.Names` -> `func (b *Bridge) Names() (string, []string, error)`
- `plan:internal/testkit/creelfake#Bridge.SendReady` -> `func (b *Bridge) SendReady(orch string) error`
- `plan:internal/testkit/creelfake#Bridge.Refuse` -> `func (b *Bridge) Refuse(orch string, err error)`
- `plan:internal/testkit/creelfake#Bridge.FailNames` -> `func (b *Bridge) FailNames(err error)`
- `plan:internal/creelengine_test#env.withBridge` -> `func (e *env) withBridge(bridge *creelfake.Bridge) *env`

**Edit:**
- `internal/creelengine#Options`
- `internal/creelengine#Run`
- `internal/creelengine#Store`
- `internal/creelengine#Store.post`
- `internal/creelengine#Store.refuseOtherHub`
- `internal/creelengine#Store.Forward`
- `internal/creelengine#Store.WaitReply`
- `internal/creelengine#Store.settledOutcomeLocked`
- `internal/creelengine#PostResult`
- `internal/creelengine#PostTags`
- `internal/creelcli#postCmd`
- `internal/creelcli#forwardCmd`
- `internal/creelengine_test#TestStore_Mail`
- `internal/creelengine_test#TestStore_InjectAndHold`
- `PATTERN.md`
- `pattern/PATTERN-agent-addressing.md`

**Extends:**
- `internal/creelengine/doc.go`

**Uses:**
- `plan:internal/agentname#ParseMachineAddress`
- `plan:internal/agentname#IsMachineAddress`
- `plan:internal/creelengine#eventMachineOutbound`
- `plan:internal/creelengine#machineOutboundEntry`
- `internal/machinename#CheckName`
- `internal/creelengine#validateBodyPresent`
- `internal/creelengine#lineOverBudget`
- `internal/creelengine#MessageLine`
- `internal/creelengine#isOrchName`
- `internal/creelengine#Store.addWaiterLocked`
- `internal/creelengine_test#newEnv`
- `internal/creelengine_test#env.withHubs`
- `internal/creelengine_test#env.start`
- `internal/testkit/creelfake#NewHubs`

**Intent:**
D1, D2 and the post half of D3: creel takes a post to a `<machine>::<name>` address and journals it as a machine-outbound message; the Slack relay alone carries it (cards 4 and 10).

`MachineBridge` is the told seam through which creel learns what it cannot know itself; its wiring in `internal/creelcli` fills it from `internal/slackengine` (card 6), since `internal/slackengine` imports `internal/creelengine`.
Its methods: `Names() (current string, former []string, err error)`, this machine's names, and `SendReady(orch string) error`, nil when orch has a slack button that is on, the machine holds both tokens and the hub its setup, and otherwise a refusal naming the way forward (the button, `lym slack login` or `lym slack setup`, or for an orch with no button, such as a pair's, posting from the hub's orch or a beam's).
`Options` gains `Bridge MachineBridge`; `Run` hands it to the store through `SetMachineBridge` beside `SetHubs`; `Store` gains the `bridge` field.

`Store.post` routes a receiver for which `IsMachineAddress` holds before the other-hub test: with `opts.Interrupt` it refuses through `refuseOtherHub`, otherwise it calls `postMachine` and answers its id and waiter.
`postMachine` refuses, in this order, each with its way forward, journaling nothing:
1. A nil bridge: `creel: this daemon was started without the Slack bridge, so %s takes no mail from here; way forward: stop this daemon so lym creel starts it again with the bridge wired`. Production sets it from card 6 on, so this is a fail-loud wiring guard; a row still reaches it through `newEnv` without `withBridge`.
2. A `Names` error, returned as it is.
3. A receiver `ParseMachineAddress(receiver, current+former, machinename.CheckName)` refuses (D1: a bad machine part, a name part that does not parse, this machine's current or a former name).
4. A sender that is not an orch strand (`sender.Kind != KindStrand || !isOrchName(sender.Name)`): `creel: only an orch strand posts to another machine, and %s is none; way forward: ask your hub's orch to post it`. This covers a program, EXTERNAL and every other strand.
5. A receiver whose role is not `orch`: `creel: only an orch of another machine takes mail across machines, and %s is none; way forward: post to <machine>::<shortname>:orch`.
6. `bridge.SendReady(sender.Name)` refusing, returned as it is (button off, token or setup missing, an orch with no button such as a pair's).
7. An empty body, or a line over `boundaryClaimBudget` as `widestMachineLine` renders it, refused with a wording of its own, since a file path does not reach the other machine: `creel: the message's line is %d bytes with the bridge envelope and stamp, over the %d-byte limit; way forward: shorten the message or split it into several posts`.
Then, under the store lock, it journals `machine-outbound` with a new id, `From` the sender, `To` the receiver as written, `ReceivingMachine` the machine part, kind mail, the body and the expect-reply mark, and answers that id; with `opts.Wait` it registers the reply wait first, as `postOtherHub` does.
Bound: this opens one edge, orch to orch across machines, and only for an orch whose button `SendReady` finds on; every other sender and receiver is refused above.

`widestMachineLine` renders the longest line the receiving creel can show for the post: `MessageLine` of a message from `FormatMachineAddress(<63-character machine>, from)` with the body, the untrusted machine stamp and the widest id, on a body led by `MachineEnvelopeWidth(from, to)` bytes, so D5's envelope and body fit the cap together.
`MachineEnvelopeWidth` returns the byte length of the longest envelope line `[lym-bridge] from=<machine>::<from> to=<to> id=<id> reply-to=<id>` takes with a 63-character machine and the widest ids; card 7's formatter test pins that its line never exceeds it (see `## Shared Decisions`).
The stamps are declared here because the width needs the longer one; card 5 shows them.

`refuseOtherHub` also refuses a name for which `IsMachineAddress` holds: `creel: %s is a name of another machine, and %s stays inside one hub; way forward: lym creel post %s "<text>"`, so `inject`, `interrupt`, `hold`, `release` and `post --interrupt` refuse it.
`Forward` refuses a receiver for which `IsMachineAddress` holds, with the same way forward, before `checkReceiverLocked`.
`WaitReply` accepts an id of `machineOutbound` posted by the caller, and `settledOutcomeLocked` answers a wait on a machine-outbound id with the oldest reply to it or, once returned, with `OutcomeReturned` and `message()`.

`creelfake.Bridge` is the test's `MachineBridge`: `NewBridge` takes the names, `Names` answers them unless `FailNames` set an error, and `SendReady` answers nil unless `Refuse` set an error for that orch.

Comments and help the new edge makes wrong change with it: `PostResult`'s `ReceiverLive` is nil for a receiver on another machine too, `PostTags` are dropped by a post to another machine too, `postCmd`'s help names `<machine>::<name>` of an orch on another machine as a receiver (orch to orch, refused with `--interrupt`), and `forwardCmd`'s help says mail from another hub or machine is forwarded only to a strand of this hub.
`env.withBridge` sets it on the store; it lives beside `env.withHubs` in `verbs_test.go`.

`TestStore_Mail` (tier 0) gains:
- a row where an orch's post to `osl::zz:orch` journals one `machine-outbound` record with receiving machine `osl`, answers its id, and a `--wait-reply` style post's wait stays open; a `WaitReply` on that id from its poster registers a wait that stays open, and one from another strand is refused as not its post;
- a table row `Post_ToAnotherMachineIsRefusedWithItsWayForwardAndJournalsNothing`, fields `{name, sender, receiver, body, refuse map, namesErr, want}`, with rows: no bridge; a `Names` error from `FailNames`; this machine's current name; a former name that fails the name rule; a bad machine part; a name part that does not parse; a program sender; an EXTERNAL sender; a non-orch strand sender; a non-orch receiver; a `SendReady` refusal from `Refuse`, returned as it is (the refusals' own words are `TestSlackBridge`'s, card 6); a line that fits locally but not with the envelope and machine stamp, naming the shorten-or-split way forward; an empty body;
- a row where `env.withHubs` and `env.start` over a pending machine-outbound message, posted before the start and after it, make no spawn and no deliver to any hub, and with the clock advanced past the hour journal no `remote-delivered` or `outbound-returned` record (a per-hub worker would return it there with `hub-unreadable`);
- the forward row: a forward to `osl::zz:orch` is refused naming `lym creel post`.
`TestStore_InjectAndHold`'s row `AnotherHubsNameIsRefusedByEveryVerbThatStaysInsideOneHub` gains `osl::zz:orch` beside the other-hub name for every verb, with nothing journaled, queued or interrupted.

`PATTERN.md`'s entry line becomes:

```markdown
- `PATTERN-agent-addressing` — Naming or messaging an agent: names only via `internal/agentname`; only through creel; across hubs with the sender confirmed, across machines orch to orch via the Slack relay; authority only for a trusted hub's or approved machine's orch. (test) — [background](pattern/PATTERN-agent-addressing.md)
```

`pattern/PATTERN-agent-addressing.md` gains an `## Across machines` rule: messaging an orch of another machine: only an orch with a slack button posts and replies, only to an orch, as a creel post to `<machine>::<name>` that creel journals and the Slack relay alone carries; mail from another machine reaches an agent only from creel's deliver verb that only `<shortname>:SLACK` may call, stamped `(other machine, …)`, with authority only for the orch of a machine the operator approved; held by the tests in `internal/creelengine` and `internal/slackengine`.
Its `## Slack boundary` rule gains that creel hands a `<machine>::` post to the relay, which alone posts it to Slack.
`internal/creelengine/doc.go`'s `# Mail across machines` gains the post rule, the refusals and the seam, in the words above.

**ImpactSummary:** `post`, `refuseOtherHub`, `Forward` and the reply wait gain a branch for `::` names only; every other name takes its old path, and `Options.Bridge` is nil until card 6 wires it, so production refuses such posts until then.

**Commit:** 3: creel journals an orch's post to another machine
