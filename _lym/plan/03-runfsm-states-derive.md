# Card 3 — runfsm-states-derive

**Create:**
- `internal/runfsm/doc.go`
- `internal/runfsm/testmain_test.go`
- `internal/runfsm/derive_test.go`
- `plan:internal/runfsm#State` -> `type State string`
- `plan:internal/runfsm#Order` -> `func Order() []State`
- `plan:internal/runfsm#Records` -> `type Records struct`
- `plan:internal/runfsm#Reading` -> `type Reading struct`
- `plan:internal/runfsm#Derive` -> `func Derive(r Records) Reading`
- `plan:internal/runfsm#TestDerive` -> `func TestDerive(t *testing.T)`
- `plan:internal/runfsm#TestOrder` -> `func TestOrder(t *testing.T)`

**Uses:**
- `internal/shedengine#Status`
- `plan:internal/shedengine#StateCancelled`
- `internal/battenrecipe#RowsBeforeRunShed`
- `internal/boardengine#IsEmptyStatus`
- `internal/boardengine#IsDoneStatus`

**Intent:** Start `internal/runfsm`, the one package that names a run's lifecycle states and derives them from tracked records alone (D1).

`State` takes the nine D1 values as constants: `landed`, `cancelled`, `cancelling`, `paused`, `halted`, `landing`, `running`, `starting`, `queued`.
`Order` returns them highest precedence first, in that order.

`Records` holds what the tracked records say of one slug, every field filled by a reader, none read here:
- `BoardFound` and `BoardStatus *string`, the slug's board entry and its status;
- `Batten *shedengine.Status`, the prime's batten record, nil when there is none;
- `Child *shedengine.Status`, the task worktree's own run record, nil when there is none;
- `TaskBranchOnOrigin`, read only for a cancelled batten record;
- `PastParentMerge`, whether origin's fetched parent branch holds the task branch's work, read only for a child at Finalize;
- `PairPresent`, whether this machine has the pair.

`Derive` walks `Order` and returns the first state whose condition holds, exactly as D1 lists them:
`landed` for a batten record `done`, or no batten record and a board status `done`;
`cancelled` for a cancel intent with a finish; `cancelling` for one without;
`paused` for a pause intent, or a batten or child shed state `paused`;
`halted` for a child `blocked`, `failed` or `awaiting`;
`landing` for a child whose current row is Finalize or whose history holds a Finalize entry;
`running` for a batten record at Run-Shed or a later row, read through `battenrecipe.RowsBeforeRunShed`;
`starting` for a batten record at an earlier row;
`queued` for a board entry with an empty status and no batten record.
A `cancelled` batten record whose task branch is gone from origin counts as no batten record.
`Reading` returns `State`, `Found` (false when no state's condition holds, as for a draft entry with no record),
`PastParentMerge` for a `landing` state, `PairKept` for a `cancelled` state whose finish kept the pair, and `PairPresent`.

`doc.go` states the package's purpose and the rule that it derives from tracked records alone, never from a `.lym` cache.
`testmain_test.go` is the package's `TestMain` through `tmuxkit.Main`, untagged.
`derive_test.go` is tier0:
`TestOrder` pins the order;
`TestDerive` is a table with one fixture per state built from records alone, one fixture per adjacent pair of states meeting both conditions reading the earlier,
a slug with no batten record and a `done` board status reading `landed`,
a released task whose cancelled record's task branch is gone reading `queued`,
a kept cancel's record on a machine without its pair reading `cancelled` with `PairKept`,
and a draft entry with no record reading not found.

**Commit:** 3: runfsm-states-derive
