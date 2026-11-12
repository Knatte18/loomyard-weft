```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism end to end on one concrete case and checked the end state against the decision's purpose.

**Per-decision walkthrough**
- D1, an upkeep pass in a task worktree reading and writing `&lt;anchor&gt;/_lym/testcover/`: purpose met. Every reader and writer named passes one maps dir: `testmap.go:262`, `run.go:83`, `redundant.go:94`, `redundancy.go:282`, `dobbydeps.go:244`.
- D2, a pair added after the prime commits maps: purpose met. The weft fork drops only the run-records root (`forkrecords.go:48`).
- D3, a task touching `internal/x` lands and the parent's pass plans at the new HEAD: purpose met. The selector diffs the map's commit against the new HEAD and queues the affected tests.
- D4, a named rebuild in the prime writes maps and then commits a positive pathspec: purpose met. A call that refuses after D11's import returns before the commit, and the record covers that through "the next call commits them".
- D5, a worktree whose only uncommitted change is under `_lym/testcover/`: purpose met. `Clean` passes the weft-exempt pathspecs to `git status` (`warpclean.go:28`).
- D6, a call in the prime on a machine that is not `prime_owner`: purpose met. The refusal reads the root's owner, and a task reads its parent's.
- D7, map P stays at H1 while the pass runs at H2, and the audit then samples a test of P: **purpose met only in outcome, mechanism gap** (F2).
- D8, a map written compact and an old indented map read back: purpose met.
- D9, an agent in a task runs the redundancy sweep's hand rebuild `lym gate testmap internal/foo`, and the pair is then torn down: **purpose missed** (F1).
- D10, a prime with no `test_map_upkeep` key: purpose met. The dobby returns at `!cfg.TestMapUpkeep` (`dobbyengine/testmap.go:51`).
- D11, the owner's first hand call with an empty `_lym/testcover/` and 211 bolt maps: purpose met. The steps run remove (none), import, plan, commit, remove. A hand call writes no exit record (`testmap.go:705`), so once the operator turns upkeep on, the dobby reads "no exit record" and starts a pass. That is harmless.

**Findings**

**F1. MEDIUM, design, D9 (`decision-record.md:103-105`) and its acceptance criterion at `:181`.**
D9 says a pass registers in runproc under the task's slug so that teardown ends it. In the code, only a `--upkeep` call registers.
- `runTestMap` calls `registerUpkeepPass` only when `request.upkeep` is set (`internal/gatecli/testmap.go:252-256`).
- `--upkeep` is a hidden flag (`testmap.go:134-137`) and refuses named packages (`testmap.go:238-239`).
- So the hand run D9 describes ("the operator or an agent runs it by hand"), and the named rebuild D9 cites, register nothing.
- `endOwnedProcesses` → `runproc.EndSparing(…, slug, …)` (`internal/pairteardown/teardown.go:697-698`) never reaches such a call. It keeps running while `RemovePair` deletes the worktree, then writes and commits into a removed weft path.
- The record does not say whether every task-worktree call registers, or that a hand call registers and ends some other way.

**F2. LOW, design, D7 (`decision-record.md:79-82`).**
D7 says an audit job "writes the planned map at HEAD … so its commit and file hashes move to HEAD". But the package job is handed the on-disk map, not the plan.
- The job reads the map file (`internal/gatecli/testmappackage.go:81-87`).
- Under D7 that file still holds H1's `Commit`, `Files`, `OtherFiles` and `Linked`.
- Only `PlanUpkeep`'s in-memory result moves those fields to the tree (`internal/testcover/upkeep.go:79-83`), and D7 leaves that result unwritten.
- D7 changes only the guard's comparison, to the planned HEAD. It does not say the job re-plans the map or is given the plan's map. Retargeting the guard alone would write H1's file hashes under H2's commit.

Tool calls:
1. Read `internal/fabricengine/forkrecords.go` and `fabricengine.Clean`: my context did not show what D2's fork drops or how D5's exemption reaches `git status`.
2. Read `structuralNeverCommittedDirs` and `lock.refuseTrackedTree`: to check where D4's map locks land when maps are written on the weft path.
