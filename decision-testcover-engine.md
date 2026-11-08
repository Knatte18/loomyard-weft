## Goal

internal/testcover: the coverage runner, content block keys and the map record, with testtiming's package-scoped verdict

Takes: cmd/testtiming's runner, profile parsing, containment and greedy pass, spawn scan and keep scan
Delivers: internal/testcover holding the moved runner, eligibility rules, spawn scan and keep scan, content block keys (D3), the per-package map record written and read through internal/state at a path internal/lymdirs names (D4); cmd/testtiming judging by the package view with a module-wide column, its flags and -pkg splice unchanged; cmd/lym guard tests re-pointed at the moved files
Exports: the engine's build, run-alone, profile parse, block key, verdict and greedy functions, the spawn and keep scans, and the map record type with its read and write

## Scope

The part changes these packages:

- internal/testcover (about 1300 changed lines)
- cmd/testtiming (about 300 changed lines)
- cmd/lym (about 10 changed lines)
- internal/lymdirs (about 10 changed lines)

## Decisions

### D1 — One engine package serves the tool, the map verb and the gate step

`internal/testcover` (new) owns:

- building a package's coverage test binary;
- running each top-level test alone, serially, with its own coverage profile;
- parsing profiles and keying blocks (D3);
- containment and the greedy removable set;
- the coverage map record (D4).

`cmd/testtiming` keeps its flags and report writing and calls the engine instead of its own copies.
The eligibility rules move with the engine unchanged: a test that is skipped, failed or covers nothing is not judged, and neither is one the static spawn scan marks as possibly running module code in another process.
The static spawn scan moves from `cmd/testtiming/spawnscan.go` into `internal/testcover` (new) with the eligibility rules.
The keep scan, which reads each `//testtiming:keep` marker and its reason and is now `scanKeeps` in `cmd/testtiming/keep.go`, moves there with it.
The tool still lists a kept candidate under Kept with its classification unchanged, while D7 step 2 drops a kept test from judging.

Rationale: three callers need the same runner and verdict,
and a `main` package cannot be imported, so the spawn scan and the keep scan leave `cmd/testtiming` with the engine.

### D2 — The tool judges by package-scoped blocks and still shows the module-wide verdict

`go run ./cmd/testtiming -redundancy` keeps one coverage binary per package, instrumented over the whole module as today.
It derives the package view by keeping only blocks in the package's own files.
Candidates and the greedy removable set come from the package view.
The report shows, for each candidate, whether it is also contained under the module-wide view; a cut contained under both is the safer one.
`-pkg` narrows the run and splices only its packages' sections, as today.

Rationale: the module-wide view counts fixture-package blocks and leaves almost every test unique, which is why `docs/benchmarks/test-redundancy.md` lists no `internal/fabricengine` candidate.
One run gives both views, because a module-wide profile holds every block of the package's own files.

### D3 — A block is keyed by content, not by position

A block's key is:

- its package-relative file name;
- its enclosing function, spelled with the receiver type for a method, or a fixed name for code outside any function;
- a hash of the block's source text;
- its ordinal among blocks with the same first three parts.

The map (D4) and the gate step (D7) compare blocks by this key.

Rationale: a profile names a block by line and column, and any edit above it shifts those.
A content key matches an unchanged block across commits by reading the files alone.
An edited block gets a new key that no map test covers, so an edit errs towards no finding.

Bound: two blocks with the same text in one function are told apart by ordinal only.
Inserting a block whose text repeats one in the same function shifts the ordinal of every later identical block,
so each judged test covering one of those blocks can get a wrong cover and a false contained verdict.
Review reading each finding against its covering tests bounds that.

### D4 — A coverage map per package, machine-local under the hub's `.lym`

One JSON record per package, at a path `internal/lymdirs` names under the hub's never-tracked `.lym` tree, shared by every worktree of the hub on the machine.
It holds:

- the commit it was built from and the time it was built;
- a content hash of each Go file the package directory tracks at HEAD, the set D5 compares and its dirty check covers;
- the block key table (D3);
- for each judged top-level test of `tier0` or `tier1`: its tier, a hash of its function's source text, its standalone duration and the indexes of the blocks it covers;
- for a package whose coverage build failed, the failure (D5).

Coverage is scoped to the package itself (`-coverpkg` names only it).
The record is written and read through `internal/state`, never refuses an unknown key, and replaces the old record whole, so a reader sees either the old map or the new one.

Rationale: each machine builds its own map from its own runs; nothing waits on another machine.

## Constraints

- Per-test coverage needs each test in its own process run: `runtime/coverage` counters are process-wide, so tests under `t.Parallel` mix their counts.
  The map cannot come from a normal tier run.
- No test of this feature launches a test binary or `go test`: the runner's tests use a stand-in `go` and fake profiles, as `cmd/testtiming`'s prebuild test does.
- `PATTERN-gate-slot`: the verb and the step run tests only while holding gate slots.
- `PATTERN-fabric-boundary`: the verb's tracked-file and status reads (D5) and the step's merge-base reads (D7 step 1) go through `internal/fabricengine`.
- `PATTERN-config-layers`: every new key has a default and loads through `configengine`; no meaning of an existing key changes, so no reinterpretation ships.
- `PATTERN-cli-cobra`: `lym gate testmap` joins the existing `lym gate` cobra subtree.
- `PATTERN-refusal-way-forward`: the verb's refusal outside the prime worktree and the unknown report name refusal each name their way forward, and a test reaches each.
- `PATTERN-code-conventions`: the step's keys come from `gate.yaml`, never a hard-coded key list.
- The prime parent's worktree is updated in place when a run lands, so its HEAD tracks the prime branch.
  `internal/fabricengine/merge.go`'s merge runs `MergeStart` in the parent's warp checkout under the weft write lock.

## Auto-mode assumptions

- The full design is built, not only the tool change: the goal asks that redundant tests stop piling up, which needs the check on new tests.

## Open risks

## Acceptance criteria

- D1: the same block sets fed to the engine's verdict give the same contained set and removable set whichever caller feeds them.
- D1: a skipped, failed, coverage-free or out-of-process test is never judged, as in today's report.
- D2: given profiles where a test's unique blocks lie only in another package, the tool lists it as a candidate and marks it not contained module-wide.
- D2: a `-pkg` run rewrites only its packages' sections of an existing report.
- D3: a block whose text is unchanged but shifted by lines above it keeps its key; an edited block gets a new key; same-named methods on two receivers get different keys.
- D4: a map written and read back yields the same tests, hashes and block sets; an unknown key in the record is kept, not refused.
