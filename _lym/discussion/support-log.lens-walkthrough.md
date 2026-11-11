```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism end to end on a concrete case, feeding each step what the step before it actually produces.

**Findings**

1. **BLOCKING, D4 (and D5's base side).** The step that keys the changed file's blocks at the base and in the working tree has no input that produces block positions.
   - Evidence: `testcover.KeyBlocks` takes `[]ProfileBlock`, and those positions come only from a coverage profile (`internal/testcover/profile.go` parses profile lines).
   - testmap gets `Map.Blocks` from the first test run's profile (`internal/gatecli/testmappackage.go`). Nothing in `internal/testcover` splits source into cover blocks.
   - So the gate has no source of blocks for the working-tree content without a coverage build of the changed package. For the base it has only the changed package's own map, and only while that map's hashes match the base.
   - The record does not decide which route to take: a cover build of each changed package per gate, a source splitter that mirrors cmd/cover's block rules, or the changed package's own map for the base side. Whichever it picks, D5 needs a matching miss, such as "the changed package's own map is not fresh against the base". This is a fact the design rests on, and it is left to Plan.

2. **MEDIUM, D2 and D5.** Block keys and file hashes are not qualified by package, but D2 records them across packages.
   - Evidence: `BlockKey.File` is "the block's file name within its package directory", and `Map.Files` is keyed by bare file name.
   - Case: test B covers `internal/a/doc.go` and B's own package has a `doc.go`. Their keys and hashes collide, so D5's per-file hash check and D4's match against the removed key compare the wrong file.
   - D2 should state that cross-package keys and hashes carry the package path. Old-format maps then need the D2 marker to tell the two shapes apart.

3. **MEDIUM, D5 file-hash rule.** Case: at the base, a covered package R gains a new file `r2.go` that holds an `init` or a method.
   - Every hash the map records still matches, so the map reads as fresh. The test path changed, but nothing widens.
   - The rule should compare the set of Go files for each recorded package as well as their hashes. The map holds that set for its own package ("each Go file the package directory tracks").

4. **MEDIUM, D3.** Case: A changes, and B and C reach A. `impactset.Targets` hands D3 a flat `ModuleTargets.Packages` list with no split between changed and importing packages.
   - D3's split therefore has no input from today's step. The mechanism has to carry the changed set out of the derivation.
   - This is plan-level, but D1's claim that `coverage` runs everything `diff` does first and then only removes holds only if that split exists.
   - Packages pulled in through `BinaryTests` (a `cmd/lym` change) are not importers of a changed package. D3 would treat them as importers, and with no changed Go package there is nothing to select, so their unrecorded tests decide what runs. They do run, because spawning tests are dropped from the maps. The result holds, but the record says nothing about this case.

5. **LOW, D3.** "Every test function of the package … that its map does not record" has to be limited to tests of the tier being run. The map records tiers 0 and 1 only, so without that limit a tier-0 run picks up the package's tier-2 tests as unrecorded.

6. **LOW, D6.** The tier-1 command is `go build ./... && go vet {targets} && go test …`, so a second invocation reruns the build and vet. Running the selected set as a second invocation of the tier command doubles that fixed cost.

7. **LOW, D8.** The claim "takes no map lock" and the torn-read case do not match the code.
   - `testcover.ReadMap` reads through `state.ReadJSON` with a lock path.
   - `WriteMap` writes atomically ("a reader sees either the old map or the new one"), so a torn map cannot occur. The acceptance criterion that tests a "partial" map file tests a state the writer never produces.

**Per decision**

- D1, `darn-write` key with `scope: coverage` under scoper `go`: met if the changed set is exposed (see finding 4).
- D2, test in B calling A: not met as written, because of the key and hash collision (finding 2).
- D3, A changed with importers B and C: the input split is missing (finding 4). Unrecorded tests must be limited to the tier (finding 5).
- D4, an edit inside function F of `a.go`: not met, because there is no producer of block keys (finding 1).
- D5, a new file in a covered package: not met (finding 3). The other cases are met.
- D6, two invocations: met, with a cost note (finding 6). Selected runs carry `-run` args, so the green record's empty-args rule already keeps them from counting as whole.
- D7: met.
- D8: met, but its stated mechanism is off (finding 7).

**Tool calls**

- Read `internal/testcover/coveragemap.go`, the head of `blockkey.go`, and `impactset.Targets`. My context had the agents' summaries, not the field shapes of `BlockKey`, `Map.Files` and `Targets`.
- Grepped testcover for block producers and testmap's `Blocks` assembly, to check whether any source-only block splitter exists. My context lacked that.
