# Card 8 — what batten's eligible-stop hold reads

**Create:**
- `plan:internal/battenshed#ChildArbiterStop` -> `type ChildArbiterStop struct`
- `plan:internal/loomengine#ArbiterTimeout` -> `func ArbiterTimeout(cfg Config) time.Duration`
- `plan:internal/loomengine#ArbiterAppliedPath` -> `func ArbiterAppliedPath(dir string) string`
- `plan:internal/arbiterstop#HoldReading` -> `func HoldReading(location *lyxcwd.Location, status shedengine.Status) (battenshed.ChildArbiterStop, error)`
- `plan:internal/arbiterstop#TestHoldReading` -> `func TestHoldReading(t *testing.T)`

**Edit:**
- `internal/loomengine#ResolveArbiter`
- `internal/loomengine#ArbiterApplied`
- `internal/loomengine#TestArbiterPaths`
- `internal/arbiterstop/doc.go`
- `internal/arbiterstop/arbiterstop_test.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/arbiterstop#Check`
- `plan:internal/arbiterstop#NewReaders`
- `internal/loomengine#LoadConfig`
- `internal/loomengine#ListArbiterAttempts`
- `internal/loomengine#ArbiterEnvelopePath`
- `internal/loomengine#ReadSettleRecords`
- `internal/loomengine#StopKeyOf`
- `internal/testkit/locationkit#Location`

**Intent:**
D9's hold needs, at a halted stop, the facts its start and end conditions name; this card adds the reading and the type it fills, and card 9 decides on it.
`battenshed` holds no copy of the eligibility rule: the reading carries `arbiterstop`'s answer as a bool.

`battenshed.ChildArbiterStop`, declared in `deps.go` beside `ChildArbiter`, holds:
`Eligible bool`, whether the arbiter verb would admit the stop;
`On bool`, whether the child's `arbiter_max_spawns` is above 0;
`Timeout time.Duration`, one arbiter spawn's resolved timeout;
`SpawnsLeft bool`, whether the run's attempt records are below `arbiter_max_spawns`;
`Attempt bool`, whether an attempt exists at the stop's key, and for the newest such attempt `AttemptDeadline time.Time`, `HandOnAt time.Time`, the write time of its `envelope.json` when that envelope carries verdict `hand-on` and zero otherwise, and `AppliedAt time.Time`, the write time of its `applied` mark and zero when absent;
`HandOverAt time.Time`, the time of the newest settle record at the stop's key when that record's action is `hand-over`, and zero otherwise.
Each field's comment says where it comes from.

`internal/loomengine`:
`ArbiterTimeout(cfg)` returns `arbiter_timeout_min` in minutes, or `arbiterDefaultTimeout` when it is 0, and `ResolveArbiter` takes its `Timeout` from it, so batten and the verb resolve one value.
`ArbiterAppliedPath(dir)` returns the applied mark's path, as `ArbiterEnvelopePath` returns the envelope's, and `ArbiterApplied` stats it.

`arbiterstop.HoldReading(location, status)` reads the task worktree at `location`:
`Eligible` is whether `Check(status, "", "", NewReaders(location))` admits the stop; batten passes no envelope kind, so a `failed` status is refused by `Eligibility`'s no-kind branch and an error stop is never eligible here, keeping D9's "error stops keep the marker hold only".
One exception keeps the hold across the arbiter's own decision: an `awaiting` stop that `Check` refuses, and whose escalation reader reports nothing pending, reads `Eligible` true when an attempt exists at `loomengine.StopKeyOf(status)` with no applied mark.
The arbiter records its circling `accept` or `continue` before a `resume`, which makes the escalation non-pending, and an attempt at the key means the verb admitted this stop; without the exception the hold would end at `since` between the verb removing the marker and the driver applying the resume.
So for an `awaiting` stop `HoldReading` lists the attempts before it decides `Eligible`.
A stop that is not eligible returns the reading with `Eligible` false and reads nothing more.
Otherwise it loads `loomengine.LoadConfig(fabricengine.ConfigDirs(location), "loom")` for `On` and `ArbiterTimeout`, lists `loomengine.ListArbiterAttempts(shedrun.ArbiterDir(location, shedrun.SelfRunID))` for `SpawnsLeft` and the newest attempt whose `StopKey` is `loomengine.StopKeyOf(status)`, reads that attempt's envelope with `loomengine.ReadArbiterEnvelope` and stats it and its applied mark for their modification times, and reads `loomengine.ReadSettleRecords(shedrun.SettleRecordsFile(loomengine.LoomFrictionDir(location)))` for the newest record at the stop key.
Any read error is returned, and card 9's caller then holds nothing.
`doc.go` says the package also reads what batten's hold decides on.

Tests:
`TestHoldReading` is unit tier over a `locationkit.Location` under `t.TempDir()`.
Its loom.yaml is `loomengine.ConfigTemplate()` written both to the board dir's `_lyx/config/loom.yaml` and the anchor's, as `internal/loomcli`'s `hubLocation` seeds it, so the strict load finds an initialized worktree.
Its escalation layout is Plan-Bouncer's run directory, `filepath.Join(loomengine.LoomReviewsDir(location), "plan")`, holding `round-1-review.md`, which makes round 1 the latest, and `round-1-escalation.md` with the frontmatter `round: 1` and the row's `cause`.
It writes attempt directories, envelopes, applied marks and settle records with `os.Chtimes` set to fixed times, and asserts the whole reading in each row:
a `blocked` stop at Plan-Write with no verify record reads eligible; the verify-halt refusal at a blocked Merge-In or Publish is `Check`'s, which the verb rows of `TestArbiterVerb_SpawnPath` reach;
an awaiting stop at a Bouncer with a pending `budget` escalation reads not eligible; one with a pending `circling` escalation and no attempt reads eligible, on, the resolved timeout and spawns left; with `arbiter_max_spawns: 0` it reads off; with the attempts at the cap and none at this key it reads no spawns left; an attempt at the key reads its deadline, a `hand-on` envelope's time and an applied mark's time, while an attempt at another key is ignored; a `resume` envelope leaves `HandOnAt` zero; a newest `hand-over` settle record at the key fills `HandOverAt`, and a newer `hand-back` at the key leaves it zero; a `failed` status reads not eligible;
at Plan-Bouncer awaiting with `round-1-escalation.md`, `round-1-circling-decision.md` in the same run directory with the frontmatter `round: 1` and `decision: continue`, which makes the escalation non-pending, and an attempt at the key whose envelope carries verdict `resume` and no applied mark, it reads eligible with `HandOnAt` and `AppliedAt` zero.
`TestArbiterPaths` asserts `ArbiterAppliedPath` names the applied file in the attempt directory.
`cmd/lyx/testdata/test-budget.yaml` raises `internal/arbiterstop` from 1 to 2.

**ImpactSummary:** loomengine gains two small accessors behind ResolveArbiter and ArbiterApplied, battenshed gains a plain reading type, and arbiterstop gains the disk reader that fills it; no caller's behaviour changes yet.

**Commit:** 8: arbiterstop: read the facts batten's eligible-stop hold decides on
