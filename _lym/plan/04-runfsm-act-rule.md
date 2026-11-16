# Card 4 — runfsm-act-rule

**Create:**
- `internal/runfsm/act_test.go`
- `plan:internal/runfsm#Answer` -> `type Answer string`
- `plan:internal/runfsm#Cause` -> `type Cause string`
- `plan:internal/runfsm#Inputs` -> `type Inputs struct`
- `plan:internal/runfsm#Aside` -> `type Aside struct`
- `plan:internal/runfsm#Verdict` -> `type Verdict struct`
- `plan:internal/runfsm#Act` -> `func Act(r Records, in Inputs) Verdict`
- `plan:internal/runfsm#TestAct` -> `func TestAct(t *testing.T)`

**Extends:**
- `internal/runfsm/doc.go`

**Uses:**
- `plan:internal/runfsm#Derive`
- `plan:internal/runfsm#Records`

**Intent:** Add D1's act rule: for one run on this machine, `act`, `wait` or `hand_over`.

`Answer` takes the constants `act`, `wait` and `hand_over`.
`Cause` names why a `wait` or `hand_over` came: push owed offline, owned elsewhere, code tip missing, hold, pause coverage, state paused, cancelling, terminal.
`Inputs` carries what the records do not hold, each zero value the benign one, so a caller that cannot fill a field yet leaves it zero:
`Here` and `Owner`, this machine and the run's effective owner, empty meaning none recorded;
`Subject`, what `lym adopt` takes to move the owner;
`Held`, a Bolt hold covers the run (D7, later parts);
`Covered`, this machine's pause record covers the run;
`PushOwedOffline`, a push is owed and origin is unreachable (D2, later parts);
`CodeTipMissing`, origin's code branch lacks the code tip the last transition names (D4, later parts);
and `Aside`.
`Aside` holds `PauseIntent`, `CancelIntent`, `Hold` and `ShedPause`, the decisions a verb sets aside when it asks.

`Act` derives the state from `r` with the asides applied:
a set-aside pause or cancel intent is dropped from the records before `Derive`, and a set-aside shed pause reads the shed state as running.
It then answers in D1's order:
`wait` for a push owed offline;
`hand_over` when `Owner` is set and differs from `Here`, a landing past its parent-side merge included;
`wait` for a missing code tip;
`act` for a `landing` past its parent-side merge with no pause coverage;
`wait` under a hold (unless set aside), pause coverage, state `paused`, `cancelling`, or a terminal state (`landed`, `cancelled`);
`act` otherwise, a run with no state included.
`Verdict` returns `Answer`, `Cause`, the derived `Reading`, and `WayForward`, one clause naming the step that ends the wait:
the same verb on the owner machine or `lym adopt <subject>` here first for `hand_over`;
`lym tender resume <slug>` for a pause; `lym batten cancel <slug>` re-run for `cancelling`;
`lym fabric remove <slug>` for a kept cancel; `lym batten cleanup --apply` for a cancel whose pair is gone; nothing to do for a landed run.

`doc.go` gains a paragraph on the act rule, its inputs, and that later parts fill `Held`, `PushOwedOffline` and `CodeTipMissing`.
`act_test.go` is tier0: `TestAct` is a table covering every answer and cause of D1's acceptance list,
`hand_over` for a landing past its merge owned elsewhere, `act` for one under a hold, `wait` for one a pause covers,
each aside lifting exactly its own wait, and each `WayForward`.

**ImpactSummary:** Adds a sentence-level paragraph to the package doc of a package no other package imports yet.

**Commit:** 4: runfsm-act-rule
