# Card 3 — pattern-coverage-map-reader

**Create:**
- `pattern/PATTERN-coverage-map-reader.md`
- `plan:cmd/lym#TestCoverageMapReaderBoundary` -> `func TestCoverageMapReaderBoundary(t *testing.T)`

**Extends:**
- `PATTERN.md`

**Uses:**
- `internal/testkit/scankit#`
- `internal/testcover#ReadMap`

**Intent:**
Declare and guard `PATTERN-coverage-map-reader`, the entry the decision record's `## PATTERN changes` adds.

`PATTERN.md` gains this line in its `## Testing` section, after `PATTERN-test-speed`:

```markdown
- `PATTERN-coverage-map-reader` — Reading the coverage maps across packages or finding a diff's changed functions: only through `internal/testreach`; only the prime's `--map` run writes its index; a report step's review note is used only when it matches HEAD's tree. (test) — [background](pattern/PATTERN-coverage-map-reader.md)
```

`pattern/PATTERN-coverage-map-reader.md` is the background, in the shape of the other background files: a `Rule:` line per clause, then why and the bounds.
- The reader: `internal/testreach` alone reads maps of more than one package and alone finds a diff's changed functions;
  the single-package readers that exist stay (the map run's own rewrite, the redundancy step's judged package and `lym gate redundant`'s package).
- The index: a cache beside the shared maps, written only by the prime's `lym gate test --map` after its maps, read by every reader that finds its stamp current, and rebuilt in memory otherwise; D1's two bounds.
- The review note: a report step's note reaches a review only when the report matches HEAD's tree, as the redundancy note does and the reach note of card 7 will.

`TestCoverageMapReaderBoundary` is a tier0 `//lym:guard`-style scan in `cmd/lym`, built on `internal/testkit/scankit` as `cmd/lym/gitrepoboundary_test.go` is.
It walks the module's non-test Go files and fails, naming file and line, when:
- a file outside `internal/testcover` and `internal/testreach` calls `testcover.ReadMap`, other than the files it pins by name:
  `internal/gatecli/redundancy.go`, `internal/gatecli/redundant.go` and `internal/gatecli/testmappackage.go`, the single-package readers;
- a pinned file no longer holds the call it is pinned for, so the list tracks the tree.

Each failure names its way forward: read the maps through `internal/testreach` (`ReadPackageMap` for one package, `Open` across packages),
or, for a new single-package reader that must call `testcover.ReadMap` itself, pin its file in the test's list with the reason.
New readers of one package's map use `testreach.ReadPackageMap`, so no later card pins a new `testcover.ReadMap` reader.
The guard's index clause comes with card 6, which pins `internal/gatecli/testmap.go` for the one `testreach.WriteIndex` call.

**ImpactSummary:** One new overview line and its background file, plus one guard test over the module's sources; no production code changes.
