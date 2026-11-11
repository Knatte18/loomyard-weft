## Goal

The coverage selector takes its target, compares own files by function, and misses on a changed environment stamp

Takes: coverage-test-select's Selector in internal/testcover as it lands, which reads the working tree only and misses on any own-file change; the map format testmap-cross-package landed (FuncKey function sets, OtherFiles, MapFormat with an EnvironmentStamp); internal/gatecli as gate-scope-coverage leaves it
Delivers: the selector with its target as an argument (working tree or a commit through fabricengine.Fabric.CodeFileAt); own non-test Go files compared by function; own non-Go files, testdata and its Go files included, a miss unless inert_paths exempts them and no go:embed pattern names them; test files compared with every top-level Test function's text removed; outcome and spawn entries read as no recorded test; a differing environment stamp a miss; the stamp holding the toolchain, GOOS/GOARCH, go.mod and go.sum hashes, the mapped tiers' command and environment digest before {parallel}, and the env.yaml variables digest; D3's map fields checked against the landed format and completed where they fall short
Exports: the selector's target argument and its verdict (affected tests or a miss with its reason), which testmap-upkeep-plan's plan and self-check call; the environment stamp builder; the MapTest outcome and spawn fields the selector reads

## Scope

The part changes these packages:

- internal/testcover (about 650 changed lines)
- internal/gatecli (about 120 changed lines)

## Decisions

### D2 — One selector serves the gate and upkeep

`coverage-test-select`'s selector is built once in `internal/testcover` with these rules beyond its D4 and D5:

- It takes its target as an argument: the working tree, or a commit read through `fabricengine.Fabric.CodeFileAt`.
- The package's own non-test Go files are compared by function, as a linked package's are,
  in place of D5's miss for any change to the package's own files.
- The package's own files other than its Go files are every tracked file under its directory tree
  down to the next package directory but the package's own Go files:
  `testdata` is included, Go files under it among them.
  A change to one that `inert_paths` does not exempt, or that a `go:embed` pattern of the package names, is a miss.
- A test file is compared with every top-level `Test` function's text removed.
  A test whose `Test` function text changed, or that is new, is affected;
  any other change to a test file, such as a helper, `TestMain`, an `init` or a package-level declaration, is a miss.
- An environment stamp that differs from the current one (D6) is a miss.
- D5's build-failure record beside test entries is no miss.
- A test whose entry records an outcome or a spawn verdict rather than a passing measurement (D4) is no recorded test.

Rationale: own non-test files are instrumented like a linked package's, so D4's function-level argument holds for them;
test files are not instrumented, so only a `Test` function's own text can be matched to one test;
a non-Go file can change any test's path, so it misses as it does for a linked package.
Under the gate, a package the diff touches still runs whole (`gate-scope-coverage`'s D3),
so the own-file rules narrow only a change that reached main after the map's `Commit`.

### D3 — Another package's coverage is recorded by function

A map records, per test, the covered blocks of its own package as today,
and for each linked package the set of functions the test covered,
each named by the linked package's import path, the file and the enclosing-function name the block keys use.
No block key of another package is stored.
A map also records the hash of each of its own package's files other than its Go files:
every tracked file under its directory tree down to the next package directory but the package's own Go files,
`testdata` included, Go files under it among them.

Rationale: the selector matches another package by function and the redundancy step judges own-package blocks only,
so another package's blocks have no consumer, and storing them grows every map and every re-stamp.

### D6 — An environment stamp replaces what the age limit stood in for

`testmap-cross-package`'s format marker also holds an environment stamp:
the Go toolchain version, `GOOS` and `GOARCH`, the hashes of `go.mod` and `go.sum`,
a digest of the mapped tiers' command and environment as configured, before the `{parallel}` substitution,
and a digest of the hub `env.yaml` variables the call passes its children.
A stamp that differs from the current one is a miss for the gate (D2) and a whole rebuild for upkeep (D1).

Rationale: a toolchain bump can move block boundaries and a module upgrade changes behaviour with no in-module file changed;
today only the age limit catches either.
A tier's parallel count changes no test's coverage, so it stays out of the digest.

### D13 — What this changes in the other entries

The three build tasks run the `darn` recipe, whose Darn-Write writes code straight from the stored record.
The parent carries each change below onto the named entry's stored record before that entry's Darn-Write writer starts,
through `lym loom decision add` on the entry's run, or by the record edit task `decision-record-replace` builds once it lands.
`testmap-backlog-drain` and `redundancy-sweep-existing` are notes with no stored record:
the parent carries each one's change below into the note's body.

- `testmap-cross-package`: D3's function sets replace block keys carrying an import path;
  the map records its own package's non-Go file hashes (D3); the format marker holds D6's stamp.
  Its criterion that a linked-only change leaves a map fresh stays, describing the interim rule (D1).
- `coverage-test-select`: the rules of D2, over D3's function sets.
  Its D5 Bound's list of what rebuilds a map becomes D1's plan verdicts at each landing and a hand call naming the package.
- `gate-scope-coverage`: in its D3 union, "every test its map does not record" includes a test whose entry records only an outcome (D4).
- `testmap-backlog-drain`: answered by D8; the note closes when D8's build lands.
- `redundancy-sweep-existing`: before judging a package the sweep runs a hand call naming it (D9), which rebuilds it whole;
  spawn-marked tests are neither candidates nor covers (D4).
- A new note, filed by this run, holds the proposal to narrow the selector's non-body miss by following the names a changed declaration is referenced by,
  with D12's miss counts as the measure of whether it pays.

The parts of this design that call the selector, D1's plan and D7's self-check, order after `coverage-test-select`.

Rationale: the three build tasks were filed on the premise that this design exists, and their records predate it.

### Added after Discussion (parent, 2026-11-11): testmap-cross-package is held until this record is approved

Decision: The run of testmap-cross-package is paused at Run-Shed, so its Darn-Write writer writes no code. It stays held until this decision record is approved and D13's changes for testmap-cross-package are carried onto its stored record.

Rationale: The operator decided in chat on 2026-11-11 to hold testmap-cross-package, coverage-test-select and gate-scope-coverage until this design is decided; the orch paused the run with lym batten pause. The board shows the old running status because the pause leaves the entry's status.

### Added after Discussion (parent, 2026-11-11): A test file's import change is no miss when the binary already links the import

Decision: The selector reads a change to a test file's imports as no miss when the import is the package's own import path, in the map's linked set, or in the standard library, for an added import; for a removed import, when it is the package's own or in the standard library. Any other added or removed import in a test file is a miss. A new or removed test file whose only imports fall under this allowance is no miss, and its Test functions are selected or dropped as D2 says for a Test function's text. This narrows D2's rule that any change to a test file outside a Test function's text is a miss: the import is the one declaration exempted. Keep the code as it stands.

Rationale: A test binary already links the package, its linked set and the standard library, so such an import adds no instrumented code and changes no recorded coverage. Without the allowance every new test file, which needs testing, would force a whole rebuild of the package, which widens the run for no gain. An import outside those sets can change what the binary links, so it still misses.

## Constraints

- PATTERN-gate-slot: every coverage build and test run of upkeep holds a slot; the plan step builds and runs nothing, so it holds none.
- PATTERN-config-layers: `test_map_max_age_days` (gateslot, a strict loader), `test_map_batch` and `test_map_every` (dobbyengine)
  leave their templates as retired keys of their modules, so `hubreconcile.Ensure`'s walk removes them from a hub base file
  and a strict load does not refuse a hub that tuned one; `test_map_every`'s 0 is carried by D8's migration.
- PATTERN-fabric-boundary: file content at a map's `Commit` and at `H` is read only through `fabricengine.Fabric.CodeFileAt`.
- PATTERN-tolerant-records: D4's entry fields, D5's failure field, D6's stamp and D12's exit record fields are added fields;
  a read refuses no unknown key, and the retired landing count stays readable in an old dobby state record.
- PATTERN-child-processes: a `--upkeep` pass stays registered as `runproc.RoleTestMap`, and a pass's test processes end through its own context.
- PATTERN-daemon-wakeups: D9's wait for a slot.

## Auto-mode assumptions

- No measurement runs before this design: each changes cost, not correctness, and D12 makes upkeep report them as it runs.
- Spawn-marked tests are never measured, hand calls included.

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 2: converged; each finding below was fixed or disputed by the round's fixer, and no fresh reviewer has seen the result; the fixer report says which.
  Review: _lym/reviews/code/round-2-review.md; fixer report: _lym/reviews/code/round-2-fixer-report.md.
  - summary-test-file-import-claim-stale (class consistency, severity MEDIUM, open in rounds 2)
  - test-file-import-allowance-unrecorded (class decision, severity MEDIUM, open in rounds 2)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria

- D2: the selector called with a commit as target gives the same verdict as with an equal working tree.
- D2: a body change to one of B's own non-test functions selects the tests that covered it and is no miss;
  an edit to a `Test` function's text selects that test alone; an edit to a helper in a test file is a miss;
  an edit to a file under B's `testdata` is a miss.
- D3: a map records, for a test of B that calls a function F of A, F under A's import path and file, and no block of A.
- D6: a map whose stamp differs in the toolchain version or the `go.mod` hash is a miss for the selector and is rebuilt whole by upkeep;
  a changed tier `parallel` value changes no stamp; a changed `env.yaml` variable changes it.
- Review: D13 names, for each of the five entries, the change it carries, and the note it files.
