# Card 13 — Master and the loom driver declare their requirements

**Create:**
- `plan:internal/websterengine_test#realMasterStarter` -> `type realMasterStarter struct`
- `plan:internal/websterengine_test#realMasterStarter.StartMaster` -> `func (s realMasterStarter) StartMaster(spec shuttleengine.Spec, gate shuttleengine.GateSpec) (websterengine.MasterHandle, error)`

**Uses:**
- `internal/loomcli#runnerDriverStarter`

**Edit:**
- `internal/websterengine#Run`
- `internal/websterengine_test#TestRun_MasterSpawn`
- `internal/websterengine_test#TestRun_RefusesBeforeSpawn`
- `internal/websterengine_test#TestRecoverSpawnOrAttach`
- `internal/loomcli#driverSpec`
- `internal/loomcli#loomCLI.startLLMDriverArm`
- `internal/loomcli#TestDriverSpec`
- `internal/loomcli#TestStartLLMDriverArm`
- `internal/sheddriver#TestTacklerSpec`
- `internal/loomcli#TestArbiterSpec`
- `internal/alignment#TestSpec`
- `internal/frictionengine#TestReflect_OneNote_SpawnsAndArchives`
- `internal/treadleengine#TestRunCircling`
- `contracts/specs/refusal-spec.md`

**Intent:**
D5 for the two single-session roles that fork and wake on their own background work, and D8's wrap at their callers.
Webster Master's spec in `Run` requires `background-wake`, forks staying implied by `ForkSubagents`: the Master stencil ends its turn after each backgrounded fork and backgrounded `recover-batch` and wakes on their completion.
`Run` passes a `StartMaster` error through `shuttleengine.ModelKey{Module: "webster", Key: "master"}.Wrap` before its existing wrap, and a requirements refusal takes that way forward in place of the `transient` one, so it names `master` and `lyx config webster`.
`driverSpec` requires `background-wake` the same way (the driver stencil runs its loop and arbiter as background jobs and wakes on them), and `startLLMDriverArm` wraps a `StartDriver` error with `ModelKey{loom, driver}`.
Every refusal still comes from shuttle's `Start`, before any pane exists, checked against the choice's engine, the default for an empty key.
The recovery strand, the tackler, the arbiter, the alignment agent, friction, merge-resolve and treadle declare nothing, which their specs keep.

`contracts/specs/refusal-spec.md` gains a webster row for Master's Start refusal (`master`, re-run the verb, or under the Webster row re-step the row) and a loom row for the driver's (`driver`, re-run `lyx loom start`), each correctness halt.

Tests:
- `TestRun_MasterSpawn` asserts the Master spec requires `background-wake`.
- `TestRun_RefusesBeforeSpawn` gains a row whose starter is `realMasterStarter`, a test starter over a real `shuttleengine.Runner` on `shuttlefake.Reed` and `shuttlefake.EngineSet` of an engine that audits forks and supports no requirement, with Master's choice engine empty: the error names engine `fake` (the default an empty key is checked against), `master` and `lyx config webster`, carries no `transient` clause, and the reed recorded no `AddStrand`.
- `TestDriverSpec` asserts the driver spec requires `background-wake`; `TestStartLLMDriverArm` gains a row whose `driverStarter` is `runnerDriverStarter` over the same kind of real `Runner`, with the `driver` key empty: the arm's error names engine `fake`, `driver` and `lyx config loom`, and the reed recorded no `AddStrand`.
- `TestRecoverSpawnOrAttach`'s first-spawn row, `TestTacklerSpec`, `TestArbiterSpec`, `alignment.TestSpec`, `TestReflect_OneNote_SpawnsAndArchives` and `TestRunCircling` assert their spec carries no `Requires`.

**ImpactSummary:** two spec builders gain one requirement and their two callers wrap the Start refusal; the declare-nothing roles change no code, only their spec tests' assertions.

**Commit:** 13: Master and the loom driver declare their requirements
