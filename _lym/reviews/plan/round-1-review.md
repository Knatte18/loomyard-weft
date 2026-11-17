---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:repo-invariants
    location: _lym/plan/06-creel-bridge-wiring.md:62
    summary: Cards 1, 4, 6, 7 and 8 create production functions whose first caller is a later card, so PATTERN-test-only-code's tier1 guard fails every batch gate that ends between the two
    fix: >-
      Add `cmd/lym#testOnlyCodeAllowed` under `**Edit:**` on each creating card and on each first-calling card, and state the entries in each card's Intent.
      Each creating card adds an allowlist entry, with a reason naming the card that adds the first caller.
      That calling card removes the entry, since the guard fails a stale entry.
      The entries, by creating card, then the card that removes them:
      card 1, `internal/agentname.ParseMachineAddress` and `internal/agentname.IsMachineAddress`, removed by card 3;
      card 4, `internal/creelengine.IsDaemonRefusal`, removed by card 10;
      card 6, `internal/creelcli.GoClient.TakeMachine`, `.ReportMachine`, `.SettleMachineDelivered`, `.SettleMachineReturned` and `.DeliverMachine`, removed by card 10;
      card 7, `internal/slackengine.FormatBridgeMessage` and `internal/slackengine.BridgeChannelName`, removed by card 10, and `internal/slackengine.ParseBridgeMessage`, removed by card 11;
      card 8, `internal/slackengine.ReadPeers`, `UpdatePeers`, `API.BotUsersNamed` and `API.UserAppID`, removed by card 9;
      `ReadBridgeMap`, `UpdateBridgeMap`, `BridgeMap.ChannelOf` and `API.PostTimestamped`, removed by card 10;
      and `API.MemberChannels` and `API.PostThreadReply`, removed by card 11.
      The target is a member glyph of a test-file var, as card 4 already lists `cmd/lym#unboundedWaitFiles`.
      `lym loom validate-plan --overlay` passed with it added on cards 1, 3, 4, 6, 7, 8, 9, 10 and 11.
  - id: F2
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/plan/04-machine-relay-verbs.md:74
    summary: IsDaemonRefusal matches only creelengine's unexported daemonRefusal, so the tier0 fakeMailbox of cards 10 and 11 cannot build the refusal their rows depend on
    fix: >-
      On card 4, add under `**Create:**`
      `plan:internal/creelengine#ErrDaemonRefusal` -> `var ErrDaemonRefusal = errors.New("creel: the daemon refused the request")`
      and `plan:internal/creelengine#daemonRefusal.Is` -> `func (e *daemonRefusal) Is(target error) bool`.
      State in the Intent that `IsDaemonRefusal(err)` is `errors.Is(err, ErrDaemonRefusal)`, and that a fake outside the package wraps `ErrDaemonRefusal` to stand for an answered refusal.
      On card 10, state that `fakeMailbox` scripts a refusal that way.
      These are the rows that need it:
      card 10, a settle answered by a refusal drops the `Accepted` entry;
      card 11, a deliver creel refuses is skipped with a notice.
      `lym loom validate-plan --overlay` passed with both Create targets added.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:guard-reach
    location: _lym/plan/11-bridge-receiving.md:80
    summary: A bridge-channel approve or revoke whose runPeerCommand fails has no stated outcome, and it runs the inbound's unlocked failingKey, episodes and answer from the bridge-poll goroutine
    fix: >-
      In card 11's `handle` step 2, state what `operator` does when `runPeerCommand` fails.
      One choice: the operator cursor stays before the message, and the failure is retried at each read under a give-up of the bridge reader's own, keyed by channel and timestamp, after which it answers in the channel and moves the cursor.
      Then add a `TestBridgeReceive` row: a `users.list` refusal on a bridge-channel `approve` is retried, and after the give-up it is answered and the cursor moves.
      Also state which goroutine owns the inbound state `runPeerCommand` touches.
      Either `runPeerCommand` takes its failure handling as an argument and touches no `inbound` field, or the bridge reader's calls into `inbound` hold a lock that card 11 names.
  - id: F4
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/plan/05-machine-inbound-deliver.md:44
    summary: Card 5 introduces the program-only deliver verb but its Intent states no bound for what the verb lets through
    fix: >-
      Add a `Bound:` line to card 5's Intent, after the `DeliverMachine` steps.
      On Linux the `SLACK` program claim is accepted from an orch strand's pane and from a process in no pane, and off Linux from any process.
      So any of them can deliver mail stamped `(other machine, trusted orch)` to an orch of this hub.
      This is creel's existing program-claim bound, which the `SLACK` claim already has, as D3 states, and the CLI has no program flag.
  - id: F5
    severity: LOW
    class: consistency
    origin: lens:repo-invariants
    location: _lym/plan/10-bridge-sending.md:48
    summary: Card 10 lists internal/slackengine/doc.go and pattern/PATTERN-daemon-wakeups.md under Extends but must rewrite existing text in both
    fix: >-
      On card 10, move `internal/slackengine/doc.go` and `pattern/PATTERN-daemon-wakeups.md` from `**Extends:**` to `**Edit:**`, dropping the then-empty `**Extends:**` label.
      In `doc.go`, the line `It runs the socket holder, the outbound read and a wake loop.` (line 79) must name the bridge poll too.
      In the PATTERN background, the Slack relay's loops bullet joins its three loops with a final `and` that a fourth loop rewrites.
      `lym loom validate-plan --overlay` passed with the move.
  - id: F6
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/plan/09-approve-and-revoke.md:39
    summary: Card 9 lower-cases the machine and then compares it exactly with this machine's names, so a former name holding an upper-case letter never gets the own-name refusal D6 asks for
    fix: >-
      In card 9's Intent, compare the command's machine with the current and every former name case-insensitively, as D6 matches the machine name.
      Add a `TestInboundRead` row: with former name `Old_Box`, `approve old_box` in an orch channel is refused naming the command for the other machine.
      Card 1 already plans that former names can fail the name rule.
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/11-bridge-receiving.md:116
    summary: Card 11 plans two rows for one behavior, a refused deliver skipped with a notice, and one of them names a refusal the relay cannot reach
    fix: >-
      Merge the row `a deliver creel refuses (a receiver that is no orch of the hub) is skipped with one notice quoting the reason, and the next message delivers` with the row `a reply-to creel refuses is skipped with a notice` into one row.
      The merged row has a reply-to creel refuses skipped with one notice quoting the reason, and the next message delivered.
      The relay only ever delivers to the channel's live local orch, so card 5's no-orch receiver refusal cannot come from it.
  - id: F8
    severity: NIT
    class: consistency
    origin: lens:guard-reach
    location: _lym/plan/05-machine-inbound-deliver.md:63
    summary: The widened releaseGoneHolder condition guards a state the existing branch already reaches, since a machine address fails isStrandName
    fix: >-
      In card 5's Intent, say that unread machine-inbound mail with no forwarder already takes `releaseGoneHolder`'s undelivered branch.
      Its sender `<machine>::<name>` fails `isStrandName` (`internal/creelengine/sweep.go:239`).
      Either drop `internal/creelengine#Store.releaseGoneHolder` from `**Edit:**` and keep the `TestSweep_RemovedHolders` row as the pin, or keep the explicit `SendingMachine` clause and say it is there for clarity.
  - id: F9
    severity: NIT
    class: consistency
    origin: lens:affected-set
    location: _lym/plan/06-creel-bridge-wiring.md:53
    summary: Card 6 sends machine mail's trust through trustField, whose doc comment still says it is the trust value of a message from another hub
    fix: >-
      Add `internal/creelcli#trustField` under card 6's `**Edit:**`.
      Reword its doc comment to cover a message from another hub or another machine.
      `lym loom validate-plan --overlay` passed with the target added.
---

### [BLOCKING] Production functions wait cards for their first caller

**Location:** `_lym/plan/06-creel-bridge-wiring.md:62`, with cards 1, 4, 7 and 8.

**Issue:** `PATTERN-test-only-code` requires every function of a production file to be reachable from a `main`.
Its guard, `TestTestOnlyCode_EveryProductionFunctionIsReachableFromAMain` (`cmd/lym/testonlycode_linux_test.go`), is a tier1 `//lym:guard` test.
The batch gate runs tiers `[0, 1]` and runs every guard test by name.
The cost batcher groups cards by budget, not by dependency (`internal/batcher/doc.go`), so a batch can end between a creating card and its consumer.
Card 6's ImpactSummary itself says its client methods have no caller until card 10, and card 7's says nothing calls its functions until cards 10 and 11.
The constraint section makes every violation of a listed PATTERN entry BLOCKING.

### [MEDIUM] No fake can produce an error that `IsDaemonRefusal` matches

**Location:** `_lym/plan/04-machine-relay-verbs.md:74`

**Issue:** `daemonRefusal` is unexported, and only `Client.do` builds it from a wire response (`internal/creelengine/client.go:274-278`).
The tier0 rows of cards 10 and 11 run over `fakeMailbox` in `slackengine_test`, which cannot build one.
Those rows are a settle answered by a refusal dropping the `Accepted` entry, and a deliver creel refuses being skipped with a notice.
The existing fakes script refusals by wrapping exported sentinels such as `ErrProgramClaimInStrandPane` (`internal/slackengine/relay_test.go:423`).
`IsDaemonRefusal` has no such sentinel, so the implementer would have to invent an export the plan does not name.

### [MEDIUM] A bridge-channel peer command that fails has no outcome, and it touches the wake loop's state

**Location:** `_lym/plan/11-bridge-receiving.md:80`

**Issue:** Card 9 sends `runPeerCommand`'s failures through `retryOrGiveUp`, which keys on the inbound's `failingKey` and calls `settle` and `answer` (`internal/slackengine/inbound.go:702-737`).
Card 11 calls the same `runPeerCommand` for a bridge-channel `approve` or `revoke` with no failure path.
So it does not say whether the operator cursor moves, losing the command, or holds with no give-up.
Card 11 also calls into `inbound` from the bridge poll's goroutine.
`wakeLoop`'s comment says one goroutine reads the channels and holds the inbound's owed reads and episodes (`internal/slackengine/relay.go:451-452`), and `failingKey` is not under `inbound.mu`.

### [MEDIUM] The deliver verb is introduced with no stated bound

**Location:** `_lym/plan/05-machine-inbound-deliver.md:44`

**Issue:** The deliver verb is the new edge through which mail reaches an orch stamped `(other machine, trusted orch)`.
The rubric's attack-surface rule has the card that introduces a verb state in its Intent what the verb lets through and what bounds it.
Card 5's Intent has no `Bound:` line.
Card 4's bound names delivery only in passing, before the verb exists.
D3 states the consequence that matters: any process that can make the `SLACK` claim can forge such mail.

### [LOW] Card 10 extends two files whose existing text it rewrites

**Location:** `_lym/plan/10-bridge-sending.md:48`

**Issue:** `internal/slackengine/doc.go:79` lists the relay's loops as the socket holder, the outbound read and a wake loop.
The bridge poll card 10 adds makes that line false.
The relay-loops bullet of `pattern/PATTERN-daemon-wakeups.md` (line 94) ends its list with `and the inbox and forwarding-file watch`, which a fourth loop rewrites.
`Extends` is for adding without changing what is there.

### [LOW] Card 9's own-name check is not case-insensitive

**Location:** `_lym/plan/09-approve-and-revoke.md:39`

**Issue:** `parsePeerCommand` lower-cases the machine name, and the own-name check then compares it with `Hub.MachineNames`.
`machinename.CheckName` admits only lower-case names, but card 1 plans a former name `Old_Box` that fails the rule.
Such a name never matches after lower-casing.
So `approve Old_Box` falls through to the name-rule refusal, where D6 asks for the own-name refusal and has the machine name matched case-insensitively.

### [LOW] Two `TestBridgeReceive` rows pin one behavior

**Location:** `_lym/plan/11-bridge-receiving.md:116`

**Issue:** Two rows pin the same branch of `handle` step 13, where creel answers a refusal and the message is skipped with a notice.
One row is `a deliver creel refuses (a receiver that is no orch of the hub) ...` and the other is `a reply-to creel refuses is skipped with a notice` (line 127).
The first row's named cause cannot occur from the relay, since steps 10 and 11 deliver only to the channel's live local orch.

### [NIT] The widened `releaseGoneHolder` guard is already covered

**Location:** `_lym/plan/05-machine-inbound-deliver.md:63`

**Issue:** `releaseGoneHolder` journals undelivered any message whose next step fails `isStrandName` (`internal/creelengine/sweep.go:239`).
A machine-inbound message's sender `<machine>::<name>` fails it, so the `SendingMachine` widening changes no outcome, and the planned row passes without it.

### [NIT] `trustField`'s comment goes stale

**Location:** `_lym/plan/06-creel-bridge-wiring.md:53`

**Issue:** `trustField`'s comment reads `the trust value of a message from another hub` (`internal/creelcli/cli.go:332`).
Card 6 routes machine mail's `trust` through it but does not list it.

## Rejected

- guard-reach finding 3, the reply-to check matching the address as written against a peer's former name: card 5 implements D8's rule as written, "only when this hub sent that id to that sender address from that receiver". Matching across a peer's names would change a Decision, which the plan cannot do on its own.

## Focus settlements

- D9: the `internal/shuttleengine/creelhooks.go` passage is `CreelTrustSentence`, which card 5 lists under `**Edit:**`, so it is carried. `orch-role.md` and `parent-directive-parent.md` are under card 5's `**Edit:**`, and `role-slack.md` under its `**Extends:**`, as D9 states.
- D3 and D10: card 4 carries the hour timer, with the post hook and the start scan's first pass arming it, and `TakeMachineOutbound` returning due messages first. Its `TestStore_MachineOutbound` row over the env's `creelfake.Clock`, with the scheduler started and no take, reaches the injected-clock criterion. Card 5's reply arms the hook too.
- D8: card 11 carries the resync hint, the holder routing of mapped and foreign channels, the per-hub limit of one per base interval, and the unpinned-bot notice. The hold stays outside `retryOrGiveUp`; only the peer-command failure path (F3) is open.
- D4: card 10 carries the send-side keys, `FindChannel` at most once per ceiling, and the archived way forward. Card 11 carries the read-side keys, and card 10 keeps the marks past a map drop.
- Card order: no card uses a symbol a later card creates. The order defect is F1's reachability gap.
- `Verify:` lines: no card has one, and no card leaves a fact to be found out while it runs.
