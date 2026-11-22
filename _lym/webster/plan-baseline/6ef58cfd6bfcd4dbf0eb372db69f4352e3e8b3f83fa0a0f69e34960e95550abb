# Card 6 — drift-autonomy

**Create:**
- `plan:internal/driftengine#ApplyAutonomy` -> `func ApplyAutonomy(cfg Config, autonomous bool) Config`
- `plan:internal/driftengine#AutonomyForcedSetting` -> `func AutonomyForcedSetting(key string) (value bool, forced bool)`
- `plan:internal/driftengine#DriftVerify.endedUnanswered` -> `func (p *DriftVerify) endedUnanswered(spawnID string) (burlerengine.ParentAsk, bool, error)`
- `plan:internal/driftengine#DriftVerify.reaskWaiting` -> `func (p *DriftVerify) reaskWaiting(spawnID string) (burlerengine.ParentAsk, bool, error)`
- `plan:internal/driftengine#DriftVerify.reaskParent` -> `func (p *DriftVerify) reaskParent(ctx context.Context, request JudgeRequest, held burlerengine.ParentAsk) (shedengine.Outcome, shedengine.OutputPointer, *judgeSession, error)`
- `plan:internal/loomcli#runAutonomy` -> `func runAutonomy(l *lymcwd.Location) bool`
- `plan:internal/driftengine#TestApplyAutonomy` -> `func TestApplyAutonomy(t *testing.T)`

**Edit:**
- `internal/driftengine#Config`
- `internal/driftengine#DriftVerify.judge`
- `internal/driftengine#DriftVerify.passCommand`
- `internal/driftengine#JudgeParentSeat.agreementGate`
- `internal/driftengine#TestParentAgreementGate`
- `internal/driftengine#TestDriftVerify`
- `internal/driftengine/doc.go`
- `internal/loomcli#driftDeps`

**Extends:**
- `pattern/PATTERN-daemon-wakeups.md`

**Uses:**
- `plan:internal/hubgeom#Autonomy`
- `plan:internal/burlerengine#ClearParentAsk`
- `internal/driftengine#haltForParent`
- `internal/driftengine#DriftVerify.runAgent`
- `internal/driftengine#DriftVerify.storeRulings`
- `internal/driftengine#DriftVerify.storeCaseRulings`
- `internal/driftengine#DriftVerify.tableWithParent`
- `internal/driftengine#DriftVerify.parentAgreement`
- `internal/driftengine#ParentAgreementGateEntry`
- `internal/driftengine#JudgeTable`

**Intent:** D3 on drift: under a run's autonomy the judge asks the parent and never waits on an operator,
and a judge ask that ends without the parent's answer stops the run instead of letting the rulings stand.

Forcing:

- `Config` gains `Autonomous bool`, the run's autonomy, which no layer spells.
- `ApplyAutonomy` returns `cfg` with `Autonomous` set to `autonomous`;
  with `autonomous` true it also sets `JudgeAskParent` true and `JudgeInteractive` false.
  No layer file is rewritten, so autonomy off restores each key's layered value at the next load.
- `AutonomyForcedSetting` answers the forced value of a drift key for the run's setting lookup:
  `judge_ask_parent` true and `judge_interactive` false, `forced` false for every other key.
- `runAutonomy(l)` is loomcli's run reader: `hubgeom.Autonomy(l, l.WorktreeName)`.
- `driftDeps`'s `config` closure returns `driftengine.ApplyAutonomy(cfg, runAutonomy(location))` after `LoadConfig`,
  so `parentAgreement` and `JudgeTable` read one forced value: the gate entry is carried, the judge spawns non-interactive,
  and its mode text is `autonomousJudgeMode`.

The ended-ask halt:

- `endedUnanswered(spawnID)` reads the seat's ask record and answers the spawn's latest ask (`LatestOfSeed(judgeAskSeed(spawnID))`)
  when it is not cleared and its state is `ParentAskTimeout`, `ParentAskNotSent` or `ParentAskNoParent`; otherwise it answers false.
  A run with no parent seat answers false.
- Both judge paths take the check: `DriftVerify.judge` under `diff: lines` and `DriftVerify.passCommand` under `diff: command`,
  which spawns the same judge through `runAgent` with the same parent agreement entry.
  Once `runAgent` returns a session and before `storeRulings` or `storeCaseRulings`, a config with `Autonomous` checks `endedUnanswered`.
  An ended ask returns `haltForParent` with a reason naming the ask, its state and reason,
  that under autonomy the rulings stand only with the parent's agreement, and the way forward `lym loom resume`, which asks the parent again over the same rulings.
  The halt comes before the rulings are stored, so judgement.yaml stays and the judge session mark stays open.
  A done ask and a config without `Autonomous` store the rulings as today.

The resume:

- `reaskWaiting(spawnID)` answers the spawn's latest ask when it is open and an earlier ask of the same spawn is `Cleared`:
  the ask an earlier resume sent and was still waiting on.
- `judge` and `passCommand`, on a session mark the earlier entry left open, with `Autonomous`, judgement.yaml present and the records' `SnapshotLost` false,
  call `reaskParent` in place of `runAgent` when `endedUnanswered` or `reaskWaiting` answers an ask for the mark's spawn.
  So no judge spawns: `runAgent` would start a fresh judge, whose spawn archives judgement.yaml and rules anew.
  The check comes before `resumeJudgeSession`, which on a lost snapshot takes a new one the stored judgement never saw;
  with `SnapshotLost` set the call runs `runAgent` as today, so a new judge rules over the new snapshot.
  On an open ask `reaskParent` clears nothing and goes straight to its wait.
  The session `reaskParent` returns goes to `storeRulings` or `storeCaseRulings` as `runAgent`'s would.
- `reaskParent` clears the ended ask with `burlerengine.ClearParentAsk`,
  then evaluates the parent agreement gate `ParentAgreementGateEntry` builds over the stored judgement, with the seat `tableWithParent` uses and no limit,
  so a new ask over the unchanged digest goes out with its deadline counted from the resume's `now`.
  While the new ask stands open it waits through `Deps.Sleep` and evaluates the gate again, as the PR-Review row's wait does:
  the wait starts at one second and doubles up to a minute while the ask record is unchanged, and returns to one second when it changes.
  A cancelled context ends the wait with its error.
  The wait can last up to `judge_parent_timeout_min`, and Drift-Verify is not among the rows the step watchdog reads as active,
  so a watchdog kill can end it: the kill leaves the new ask open and the session mark open,
  and the re-entry, finding that ask open and not ended, continues the wait over it instead of spawning a judge.
- `pattern/PATTERN-daemon-wakeups.md` gains a bullet for this wait in `internal/driftengine/verify.go` beside the drift capture waits:
  it waits from one second, doubling to a minute while the ask record is unchanged.
  - The gate passing on a done ask returns a `judgeSession` holding the stored judgement, and `judge` stores the rulings.
  - A done ask whose report holds findings runs `runAgent` as today, since only a judge rewrites rulings.
  - An ask that ends unanswered again returns the same halt.
- `agreementGate` treats a held ask that is `Cleared` as no ask: `held` false and `round` the ask's round plus one, ahead of its digest check.
  Today a new ask goes out only for new rulings; this is the one other case.
- A cause that persists halts again at the row, which the orch counts toward D5's class 5.
- `doc.go`'s judge paragraphs state the forcing of `judge_ask_parent` and `judge_interactive` under the run's autonomy,
  that under it an ask ended timeout, not-sent or no-parent halts Drift-Verify instead of passing the parent agreement entry,
  and that the resume asks the parent again over the same rulings.

Tests, tier0:

- `TestApplyAutonomy`: true forces the two keys from layered `judge_ask_parent: false` and `judge_interactive: true` and sets `Autonomous`;
  false leaves both layered values and `Autonomous` false;
  `AutonomyForcedSetting` answers the two keys forced and `judge_model` not forced.
- `TestDriftVerify`, which drives `DriftVerify.Call` through the judge, gains rows with `Config{Autonomous: true, JudgeAskParent: true}` and a judge that ends Done:
  - the spawn's ask ended timeout past `judge_parent_timeout_min`, not-sent and no-parent each end the call Stuck with a reason holding `lym loom resume` and the state,
    store no rulings, keep judgement.yaml and leave the session mark open;
  - a done satisfied ask stores the rulings;
  - with `Autonomous` false, each ended state stores the rulings as today;
  - a second call after the timeout halt clears the ended ask, records a new ask of the next round over the same digest, starts no seat,
    and, with the ask answered satisfied from the row's `Sleep` hook, stores the stored judgement's rulings;
  - a second call whose new ask ends unanswered halts again;
  - a call whose record holds a cleared ask and an open ask after it, as a step-watchdog kill mid-wait leaves it, starts no seat, records no further ask,
    and stores the stored judgement's rulings once the open ask is answered satisfied from the `Sleep` hook;
  - a second call after the timeout halt with `SnapshotLost` set starts the judge seat as today;
  - under `DiffCommand`, a timeout-ended ask halts the call Stuck with no rulings stored,
    and a second call clears it and records a new ask over the same digest with no seat started.
- `TestParentAgreementGate` gains a row whose held ask is ended timeout and cleared over the current digest:
  the gate records a new ask of round 2 whose deadline is the row's `now` plus the wait, and posts its line.

**ImpactSummary:** Drift's resolved config gains one field read by Drift-Verify's two judge paths, `judge` and `passCommand`, which each gain one halt and one resume path under it; PATTERN-daemon-wakeups lists the resume's wait; loomcli's drift wiring is the one config reader changed, and without autonomy every path behaves as today.
