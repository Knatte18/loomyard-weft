```yaml
lens: walkthrough
outcome: reported
```

Lens: walkthrough. For each decision I ran one concrete case through its mechanism step by step and checked the end state against the decision's stated purpose. I made no tool calls.

**Decisions walked**

- **D1:** `darn-write` with `scope: coverage` under scoper `go`. Today `withGateScope` only switches on `all` and `diff` (`internal/gatecli/scope.go:111-117`), so a `coverage` case is new plan work. The mechanism, "derive as `diff`, then only remove", composes. Purpose met.
- **D2:** a test of B calls A, and testmap runs with coverpkg {A, B}. The profile carries A's blocks, and the keys need A's own directory per block. Today `KeyBlocks` reads every block from one `dir` (`internal/testcover/blockkey.go:81-95`), and the record's import-path key covers that. Spawn verdict: B's test calls `A.Start()`, and A runs the `lym` binary through `os/exec`. The scan returns an empty verdict for any call through an import (`internal/testcover/spawnscan.go:256-259`). The test is recorded as in-process, and its map omits the code the child process ran. Purpose broken (finding 3).
- **D3:** A changed, B imports A, and B has a guard test G, an unmapped test U and a mapped test T covering `A.F`. The union is {T if F changed, U, G}. Purpose met, apart from finding 2.
- **D4:** an edit inside the generic method `(*Store[T]).Get`, and an edit inside a closure in `F`. `funcKeyName` and `receiverTypeName` give `Store.Get`, and closures take their outer `FuncDecl`'s name (`blockkey.go:175-207`). Naming matches, so the premise holds. The base-content read does not exist on the path the record names (finding 4). Two `init` functions in one file share the key name `init`, so pairing base to working tree by name is undefined (finding 5).
- **D5:** two cases.
  - `webster-fix` after `webster` passed tier 0 at branch commit C1, and C1 changed A. The tier base is C1, B's map was built on main, and A's hashes at C1 differ, so B runs whole.
  - `darn-write` with merge base M0. A landing on main touches `internal/configreg`, and testmap rebuilds B's map at M2. Every hash of `configreg` at M0 differs, so B misses.

  Purpose missed (finding 1).
- **D6:** tier 1 runs A whole plus B and C with `-run ^(T1|T2)$`. `recordPass` stamps every invocation with `run.args` (`internal/gatecli/green.go:447`), and `invocation` has no args field (`internal/gatecli/tiers.go:33-41`). Today's mechanism would therefore record B and C as package passes with empty args. The record names the right end state, the entry carrying its `-run` args, so this is plan plumbing. Purpose met as stated.
- **D7:** report for {A changed, B selected 2 of 9, C miss}. Purpose met.
- **D8:** `ReadMap` takes a lock file beside the map (`internal/testcover/coveragemap.go:61-64`), and a failed read is a miss. Purpose met.

**Findings**

1. **BLOCKING, design. D5, compared with Goal and D2 (the tier base).** Map freshness is checked against the tier's base, but maps are built only at prime's main HEAD. So two common cases always miss:
   - a green-ancestor base on the branch that already changed the package, which is the usual `webster-fix` case after a partly green `webster`;
   - any worktree whose merge base predates a landing that touched a widely imported package. In the week to 2026-11-11, 52 of 163 main commits touched `configreg`, `loomengine`, `gateslot`, `fabricengine` or `state`.

   The end state is the import scope the board names as the problem. Evidence: `internal/gatecli/green.go:202-221` (`newestWholeAncestor` picks a branch commit), `internal/gatecli/testmap.go:142-147` (maps are built only in the prime) and `git log --since=2026-11-04 main`. A fix within the design: D4 and D5 diff from the map's recorded `Commit` (`internal/testcover/coveragemap.go:27`) rather than from the tier base.
2. **MEDIUM, design. D5, compared with D2.** A drift in the importing package's own files between the map build and the base is not a miss. Case: B's test T is edited on main after B's map was built and now calls `A.G`. The diff changes `A.G`, and T's map entry does not cover it. T runs only if B's own `Files` are compared. D5's "for a covered package" reads, through D2's "per covered package other than its own", as excluding B. `MapTest.FuncHash` exists, but D3 never uses it. Evidence: `internal/testcover/coveragemap.go:31,46` and `decision-record.md` D2 and D5.
3. **MEDIUM, design. D2 and D3 (spawn verdict).** The spawn scan does not follow calls into other in-module packages; such a call returns an empty verdict. So a test of an importing package whose callee in another package starts a process that runs module code is mapped as in-process, with only partial coverage. When the changed code runs in the child, the test is not selected, and D1's and D5's bounds do not name this. Evidence: `internal/testcover/spawnscan.go:256-259` and `ScanSpawns`' doc bound at `spawnscan.go:175`.
4. **MEDIUM, consistency. Constraints, PATTERN-fabric-boundary bullet, compared with D4.** The bullet says to read base file content "through the path the impactset diff already uses". That path is `gitexec.Run` with `git diff --name-status` and `ls-files`, which reads no file content. The content read that exists is `fabricengine.Fabric.CodeFileAt`. As written, the step's input has no source. Evidence: `internal/impactset/targets.go:109,118` and `internal/fabricengine/codereads.go:27`.
5. **LOW, design. D4 (duplicate function names).** A file may declare several `init` functions, and all key as `init`. "Present in both whose body text differs" has no defined pairing when the base and the working tree hold different numbers of `init`s or reorder them. The selection stays a superset only if every same-named body counts as changed whenever any differs. Evidence: `internal/testcover/blockkey.go:185-190`.

Tool calls: none.
