# Coverage selector: a target, own files by function, a stamp miss

`testcover.Selector` is the one selector the coverage gate scope and the coming upkeep plan both call.
This change gives it the rules that upkeep needs on top of the gate's.

## What changes

**A target argument.**
`NewSelector(config, target)` takes a `testcover.Target`.
An empty `Commit` selects the working tree under `WorktreeRoot`, and a set `Commit` selects that commit, read through the `CommitReader`, which is `fabricengine.Fabric` in production.
`Target.Files` lists the target's files and replaces `SelectorConfig.WorkingFiles`.
Every read of the target's side goes through one `readTargetFile`, so a commit and an equal working tree give the same verdict.
The test runs every row against both targets.

**Own non-test Go files are compared by function.**
D5's miss for any change to the package's own files is gone.
The package's own files go through the same per-file comparison as a linked package's: function changes, import rules, method set, init, predeclared names and directives.
A changed own function selects each recorded test whose own-package block keys name that file and function.
Own non-Go files, testdata and the Go files under it included, miss unless `inert_paths` matches them and no `go:embed` pattern of the package names them.
That rule is unchanged.

**Own test files are compared with their Test functions set apart.**
A changed own `_test.go` file is parsed on both sides.
Its top-level Test functions are set apart, and the rest of the file is compared, ignoring comments and formatting.
A Test function whose declaration changed, or that is new, is affected whatever the map records for it.
Any other difference is the new miss `MissTestFile`: a helper, `TestMain`, an `init`, an import or a package-level declaration.
A removed Test function is neither.
A linked package's test files are still skipped.

**Outcome and spawn entries are no recorded test.**
`MapTest` gains `Outcome` (`TestOutcome`: failed, timed out, skipped, covered nothing), empty for a passing measurement.
`Selection.Recorded` now holds only passing entries.
An entry with an outcome, or with a spawn verdict marking it out of process or unresolved, is left out.
It is never selected through coverage.
`gatecli`'s `coverageUnion` therefore runs every test that `Recorded` lacks and no longer checks spawn marks itself.
For spawn-marked tests the gate runs exactly what it ran before.

**A differing environment stamp is a miss.**
`SelectorConfig.Environment` carries the current stamp without its module file hashes.
The selector fills those from the target's go.mod and go.sum through the new `testcover.StampModuleFiles`.
Any difference from the map's `Format.Environment` gives `MissEnvironment`, with the differing fields named.
In `gatecli`, `stampEnvironment` builds the two digests, over the hub's mapped tiers' commands and env (before `{parallel}`) and over the env.yaml variables.
`mappedTierConfigs` lists those tiers.
`lym gate testmap` and the coverage scope's `openSelector` both stamp through these two functions and `StampModuleFiles`, so a map's stamp and the gate's agree.
The gate reads the toolchain through `Toolchain.GoEnvironment` under `gateEnv`, as testmap does, and keeps the hub env on `testRequest.hubEnv`.

`MissOwnFiles` is removed.
`mapMiss` now also rejects a test whose block index lies outside the block table, since own-file selection reads `Blocks`.

## Design decisions

- The rest of a test file is compared syntactically, ignoring comments and formatting, rather than as raw text.
  This matches how every other Go file is compared, and comments do not change a test binary.
  A Test function is compared the same way.
- Imports in a test file get no linked-package allowance: adding or removing one is a miss, as the record lists "any other change".
  A new test file that only declares Test functions selects them.
  One with imports misses.
- D3 was checked against the landed format: function sets keyed by import path, file and function; no block of another package; `OtherFiles` with testdata.
  Nothing fell short, so the map shape and `MapFormatVersion` are unchanged.
  `Outcome` is an added `omitempty` field that old maps read as a passing entry, and every old map holds only passing entries.
- D5's separate failure field beside test entries belongs to `testmap-upkeep-plan`, which writes it.
  The selector ignores unknown keys, so that field will be no miss.
  A failure-only map with `BuildFailure` still misses, as the sibling record keeps it.
- Only `Outcome` is added to `MapTest`.
  The per-entry commit and time that D4 also names are left to their writer, `testmap-upkeep-plan`, since the selector reads neither.
