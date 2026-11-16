# Card 2 — runfsm-package

**Create:**
- `internal/runfsm/doc.go`
- `plan:internal/runfsm#State` -> `type State string`
- `plan:internal/runfsm#StateNone` -> `const StateNone State = ""`
- `plan:internal/runfsm#StateLanded` -> `const StateLanded State = "landed"`
- `plan:internal/runfsm#StateCancelled` -> `const StateCancelled State = "cancelled"`
- `plan:internal/runfsm#StateCancelling` -> `const StateCancelling State = "cancelling"`
- `plan:internal/runfsm#StatePaused` -> `const StatePaused State = "paused"`
- `plan:internal/runfsm#StateHalted` -> `const StateHalted State = "halted"`
- `plan:internal/runfsm#StateLanding` -> `const StateLanding State = "landing"`
- `plan:internal/runfsm#StateRunning` -> `const StateRunning State = "running"`
- `plan:internal/runfsm#StateStarting` -> `const StateStarting State = "starting"`
- `plan:internal/runfsm#StateQueued` -> `const StateQueued State = "queued"`
- `plan:internal/runfsm#Order` -> `func Order() []State`
- `plan:internal/runfsm#Records` -> `type Records struct`
- `plan:internal/runfsm#Derive` -> `func Derive(r Records) State`
- `plan:internal/runfsm#Answer` -> `type Answer string`
- `plan:internal/runfsm#AnswerAct` -> `const AnswerAct Answer = "act"`
- `plan:internal/runfsm#AnswerWait` -> `const AnswerWait Answer = "wait"`
- `plan:internal/runfsm#AnswerHandOver` -> `const AnswerHandOver Answer = "hand_over"`
- `plan:internal/runfsm#Reason` -> `type Reason string`
- `plan:internal/runfsm#ReasonPushOwed` -> `const ReasonPushOwed Reason = "push_owed"`
- `plan:internal/runfsm#ReasonOwnedElsewhere` -> `const ReasonOwnedElsewhere Reason = "owned_elsewhere"`
- `plan:internal/runfsm#ReasonCodeTip` -> `const ReasonCodeTip Reason = "code_tip"`
- `plan:internal/runfsm#ReasonHold` -> `const ReasonHold Reason = "hold"`
- `plan:internal/runfsm#ReasonPause` -> `const ReasonPause Reason = "pause"`
- `plan:internal/runfsm#ReasonPaused` -> `const ReasonPaused Reason = "paused"`
- `plan:internal/runfsm#ReasonCancelling` -> `const ReasonCancelling Reason = "cancelling"`
- `plan:internal/runfsm#ReasonLanded` -> `const ReasonLanded Reason = "landed"`
- `plan:internal/runfsm#ReasonCancelled` -> `const ReasonCancelled Reason = "cancelled"`
- `plan:internal/runfsm#Asides` -> `type Asides struct`
- `plan:internal/runfsm#Inputs` -> `type Inputs struct`
- `plan:internal/runfsm#Verdict` -> `type Verdict struct`
- `plan:internal/runfsm#Act` -> `func Act(in Inputs) Verdict`
- `plan:internal/runfsm#TestDerive` -> `func TestDerive(t *testing.T)`
- `plan:internal/runfsm#TestOrder` -> `func TestOrder(t *testing.T)`
- `plan:internal/runfsm#TestAct` -> `func TestAct(t *testing.T)`
- `plan:internal/runfsm#TestLeafInvariant_AllowlistOnly` -> `func TestLeafInvariant_AllowlistOnly(t *testing.T)`

**Extends:**
- `pattern/PATTERN-leaf-packages.md`

**ImpactSummary:** a new standard-library-only package `internal/runfsm` declares the run states, their order and the act rule; nothing calls it yet.

**Intent:**
Add the leaf package `internal/runfsm` that declares D1's run lifecycle states, their order and the act rule.
It imports only the standard library, so it holds no row names, shed state literals or record paths:
its callers turn tracked records into the plain facts of `Records`, and the reader that does it comes on a later card.
`doc.go` documents only this package: the states, the order, the act rule and the leaf limit.

`State` and its constants are D1's states; `StateNone` is a slug no state matches, such as a draft task with no record.
`Order` returns the states highest precedence first: landed, cancelled, cancelling, paused, halted, landing, running, starting, queued.

`Records` holds the facts `Derive` reads, every one a bool:
`Released` (a task entry with an empty board status), `BoardDone`,
`Batten` (a batten record stands), `BattenDone`, `BattenPaused`, `BattenAtRunShed` (the record at Run-Shed or a later row),
`CancelIntent`, `CancelFinished`, `CancelKept`, `TaskBranchGone` (the task branch is gone from origin), `PauseIntent`,
`Child` (a child record stands), `ChildPaused`, `ChildHalted` (blocked, failed or awaiting), `ChildAtFinalize` (at Finalize or a later row).
`Derive` takes the first state in `Order` whose condition holds:
landed for `BattenDone`, or no batten record and `BoardDone`;
cancelled for a finished cancel intent; cancelling for an unfinished one;
paused for a pause intent or a paused batten or child record; halted for `ChildHalted`; landing for `ChildAtFinalize`;
running for `BattenAtRunShed`; starting for any other batten record; queued for `Released` with no batten record.
A finished cancel whose `TaskBranchGone` holds counts as no batten record, so a released task with one reads queued and an unreleased one `StateNone`.

`Inputs` carries the act rule's inputs: `State`, `PauseIntent`, `ShedPaused`, `Owner` (the machine that owns the run, empty when this one does),
`PushOwedOffline`, `CodeTipMissing`, `PastParentMerge`, `HoldCovers`, `PauseCovers` (this machine's pause record) and `Aside`.
`PushOwedOffline`, `CodeTipMissing` and `HoldCovers` are filled by later parts; until then callers leave them false.
`Asides` names what the asking verb sets aside: `Pause` (the run's own pause intent and pause record coverage), `Cancel` (the cancel intent, finished or not), `Hold` and `ShedPause` (a paused shed state).
`Verdict` holds the `Answer`, the `Reason` of a wait or hand-over, and the `Owner` of a hand-over.
`Act` answers in D1's order:
wait for an owed push offline; hand_over when `Owner` is set; wait when the code tip is missing;
act for a landing past its parent-side merge that no pause covers, pause meaning the pause record or the pause intent unless set aside;
wait for a hold, a pause, the paused state (unless each of its causes, the intent and the shed pause, is set aside), cancelling or cancelled (unless the cancel is set aside), or landed;
act otherwise, `StateNone` included.

`TestDerive` reads every state from a fixture of facts and every pair of states whose conditions overlap as the earlier one, including no batten record with a done board reading landed and a finished cancel with its task branch gone reading queued.
`TestOrder` pins the order.
`TestAct` covers every rule and every aside, including a landing past its merge under a hold answering act unless a push is owed offline or a pause covers it, and a hand-over of a landing past its merge.
`TestLeafInvariant_AllowlistOnly` calls `scankit.AssertImportAllowlist` with no allowed import, as `internal/haltnote`'s leaf test does.
Every test is tier 0.

In `pattern/PATTERN-leaf-packages.md`, add an `internal/runfsm` section: it imports only the standard library, and its callers read the records.

**Commit:** 2: runfsm declares the run states, their order and the act rule
