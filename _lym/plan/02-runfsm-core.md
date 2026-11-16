# Card 2 — runfsm-core

**Create:**
- `internal/runfsm/doc.go`
- `internal/runfsm/state.go`
- `internal/runfsm/act.go`
- `internal/runfsm/record.go`
- `internal/runfsm/state_test.go`
- `internal/runfsm/act_test.go`
- `plan:internal/runfsm#State` -> `type State string`
- `plan:internal/runfsm#Landed` -> `const Landed State = "landed"`
- `plan:internal/runfsm#Cancelled` -> `const Cancelled State = "cancelled"`
- `plan:internal/runfsm#Cancelling` -> `const Cancelling State = "cancelling"`
- `plan:internal/runfsm#Paused` -> `const Paused State = "paused"`
- `plan:internal/runfsm#Halted` -> `const Halted State = "halted"`
- `plan:internal/runfsm#Landing` -> `const Landing State = "landing"`
- `plan:internal/runfsm#Running` -> `const Running State = "running"`
- `plan:internal/runfsm#Starting` -> `const Starting State = "starting"`
- `plan:internal/runfsm#Queued` -> `const Queued State = "queued"`
- `plan:internal/runfsm#Order` -> `var Order []State`
- `plan:internal/runfsm#CancelIntent` -> `type CancelIntent struct`
- `plan:internal/runfsm#CancelFinish` -> `type CancelFinish struct`
- `plan:internal/runfsm#PauseIntent` -> `type PauseIntent struct`
- `plan:internal/runfsm#Intents` -> `type Intents struct`
- `plan:internal/runfsm#RunRecord` -> `type RunRecord struct`
- `plan:internal/runfsm#ParseRunRecord` -> `func ParseRunRecord(data []byte) (RunRecord, error)`
- `plan:internal/runfsm#Records` -> `type Records struct`
- `plan:internal/runfsm#Derive` -> `func Derive(r Records) State`
- `plan:internal/runfsm#Records.PastParentSideMerge` -> `func (r Records) PastParentSideMerge() bool`
- `plan:internal/runfsm#Answer` -> `type Answer string`
- `plan:internal/runfsm#AnswerAct` -> `const AnswerAct Answer = "act"`
- `plan:internal/runfsm#AnswerWait` -> `const AnswerWait Answer = "wait"`
- `plan:internal/runfsm#AnswerHandOver` -> `const AnswerHandOver Answer = "hand_over"`
- `plan:internal/runfsm#Aside` -> `type Aside struct`
- `plan:internal/runfsm#Inputs` -> `type Inputs struct`
- `plan:internal/runfsm#Verdict` -> `type Verdict struct`
- `plan:internal/runfsm#Act` -> `func Act(r Records, in Inputs) Verdict`
- `plan:internal/runfsm#Verdict.Refusal` -> `func (v Verdict) Refusal(slug, verb string) error`
- `plan:internal/runfsm#TestDerive` -> `func TestDerive(t *testing.T)`
- `plan:internal/runfsm#TestParseRunRecord` -> `func TestParseRunRecord(t *testing.T)`
- `plan:internal/runfsm#TestAct` -> `func TestAct(t *testing.T)`
- `plan:internal/runfsm#TestVerdict_Refusal` -> `func TestVerdict_Refusal(t *testing.T)`

**Uses:**
- `internal/shedengine#State`
- `plan:internal/shedengine#StateCancelled`

**Intent:**
Create `internal/runfsm` (new), the one package that names a run's lifecycle states, their order and the act rule (D1).
This card holds the pure core: it reads no file and runs no git, so every rule is pinned by tier0 tests; card 3 adds the reader.

`state.go`: `State` and its nine values; `Order` lists them highest precedence first, landed, cancelled, cancelling, paused, halted, landing, running, starting, queued, and `TestDerive` pins that order.
`Derive` takes the first state whose condition holds over `Records`, exactly as D1 lists them:
`landed` for a batten record whose shed state is `done`, or no batten record and a board status `done`;
`cancelled` for a cancel intent with its finish; `cancelling` for a cancel intent without one;
`paused` for a pause intent, or a shed state `paused` on the batten or the child record;
`halted` for a child record `blocked`, `failed` or `awaiting`;
`landing` for a child record at the `Finalize` row or past it; `running` for a batten record at the `Run-Shed` row or past it;
`starting` for a batten record before `Run-Shed`; `queued` for a released task with an empty board status and no batten record.
A record is "at a row or past it" when its current row is that row or its history names it, the reading `fabricengine`'s replay already uses for Finalize; the two row names are one named constant each in `state.go`.
A batten record whose cancel intent has a finish and whose task branch is gone from origin counts as no batten record, so such a released task reads `queued` (D1).
`Records` carries the batten `*RunRecord` (nil when absent), the child `*RunRecord` (nil when absent), the board status, whether the task is released here, whether the task branch is on origin, and whether origin's fetched parent branch holds the task branch's work.
`PastParentSideMerge` is true for a child record at `Finalize` or past it whose work origin's fetched parent branch holds; it ignores the state's precedence, so a shed-paused run past its merge still finishes (see Shared Decisions).

`record.go`: `RunRecord` holds a status record's shed state, current row, the rows its history names, and its `Intents`.
`Intents` is the pair of keys `cancel_intent` and `pause_intent` that live in the batten record beside the shed-owned keys; `CancelIntent` holds when the cancel started, the machine, and remove or keep, plus `*CancelFinish` with the time it ended and whether the pair was removed; `PauseIntent` holds who paused, the machine and when.
`ParseRunRecord` decodes the bytes of a `status.json` read from disk or from a git tree tolerantly, as `shedengine.ParseStatusTolerant` does: no unknown key and no state value is refused (PATTERN-tolerant-records).

`act.go`: `Act` answers `act`, `wait` or `hand_over` in D1's order, first match wins:
`wait` when `Inputs.PushOwed` and `Inputs.OriginUnreachable`;
`hand_over` when `Inputs.OwnedElsewhere`, even for a run past its parent-side merge;
`wait` when `Inputs.CodeTipMissing`;
`act` when `PastParentSideMerge` holds and no pause covers the run;
`wait` when a hold covers it, a pause covers it, or the state is `paused`, `cancelling`, `landed` or `cancelled`; `act` otherwise.
Every `Inputs` field's zero value is the neutral reading (no push owed, owned here, code tip present, no hold, no pause), so a caller that cannot fill a field yet, the hold and code-tip inputs later parts fill, changes no answer.
A pause covers the run when `Inputs.PauseCovers` (this machine's pause record covers the slug) or the run's pause intent stands.
`Aside` names what the asking verb sets aside before the rule runs: `OwnPause` (the pause intent and this machine's own-scope pause entry, for `lym tender resume`), `Cancel` (the cancel intent without a finish, for `lym batten cancel`), `Hold` (for `lym batten cancel`) and `ShedPause` (a shed state `paused`, for `lym batten run`, `lym loom resume`, `lym batten cancel` and `lym tender resume`, per the added decision).
`Verdict` carries the answer, the state, the reason the rule matched, and the owner and adopt subject for a hand-over.
`Verdict.Refusal` returns nil for `act`, and otherwise an error naming the slug, the state and one way forward per reason: re-run `<verb>` once origin answers; run `<verb>` on the owner machine, or `lym adopt <subject>` here first; wait for the owner's code push; `lym tender resume <slug>` for a pause; `lym batten run <slug>` for a shed pause; `lym batten cancel <slug>` to finish a cancel; nothing is left to do for `landed`; `lym fabric remove <slug>` for a kept `cancelled` run.
The text never names `lym adopt --take-hold` (PATTERN-refusal-way-forward); `TestVerdict_Refusal` reaches every reason.

`doc.go` documents the package: the states, the act rule, that its core reads no file, and that the board's `<state> · <row>` status is display only.
The package imports the standard library only in this card; card 3's reader adds `lymcwd`, `shedrun`, `shedengine`, `boardengine`, `hubgeom`, `state` and `fabricengine`.

`internal/runfsm/state_test.go` holds tier0 tests: `TestDerive`, one row per state from records alone, rows meeting two states' conditions reading the earlier one, the cancelled record with its task branch gone reading `queued`, a kept cancel with no child record reading `cancelled`, and `TestParseRunRecord`, with an unknown key and a record carrying both intents.
`internal/runfsm/act_test.go` holds tier0 tests: `TestAct` covers each line of the rule and each aside, including `hand_over` for a run past its parent-side merge owned elsewhere, `act` for one past its merge under a hold, `wait` for one past its merge under a pause or with a push owed offline; and `TestVerdict_Refusal`.

**Commit:** 2: runfsm names a run's states, derives one from its records and answers the act rule
