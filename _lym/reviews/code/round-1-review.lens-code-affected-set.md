```yaml
lens: code-affected-set
outcome: reported
```

The fork covered the code-affected-set lens.

**Affected set, per change**

1. **Removed field `HandOverSeam.DirtyPaths`** (`internal/sheddriver/drive.go:143-153`):
   - The only production writer was `internal/loomcli/drivewiring.go:306`, removed in the diff.
   - The only test writer was `drive_test.go:870`, also removed.
   - Nothing else reads the field. `go build` and `go vet` on both packages passed.
   - Covered.
2. **Removed `dirtyPathsOutsideRecords`** (`drivewiring.go`):
   - It had no other caller.
   - The imports `fabricengine`, `filepath`, `loomengine` and `lymcwd` are all still used in the file (`drivewiring.go:97,98,165,221,236,242,397,413-415`).
   - `loomengine.PathsOutsideRunRecords` keeps its other caller at `internal/battencli/wire.go:1000`.
   - Covered.
3. **Removed `strings` import** (`drive.go`): the build passes, so no use remains. Covered.
4. **Removed hand-back messages** "the tree is not clean for a hand-over", "…uncommitted paths outside the run's records, so no successor is spawned" and "the hand-over could not check the tree":
   - A search of the repo and `_bolt/_lym/specs` found no test, doc, stencil, spec or fixture that still asserts them.
   - Covered.
5. **Changed behaviour: hand-over runs over uncommitted paths.** Readers and descriptions of it:
   - `internal/loomcli/doc.go:646-649`: updated, covered.
   - The `handOver` doc comment (`drive.go:345-353`): updated. It still says "A failed spawn leaves the run parked … for the parent or a watching batten to pick up", which no longer fully holds; see F1.
   - `internal/sheddriver/doc.go:79,99`: neither line states a clean-tree rule. Covered.
   - `pattern/PATTERN-shed-driver.md:31`: names only the `HandOver` group. Covered.
   - No PATTERN entry or `pattern/` background states a clean-tree hand-over.
   - Batten's binary re-step (`internal/battenshed/innerrun_binaryrestep.go:121-130`) keeps its own dirty-tree refusal. Not updated; see F1.
6. **Test row rename and removal** (`drive_test.go:574, 588-592`): the table is the only reader, and the redundancy report lists `TestGoDriver_Settle` as unique. Covered.
7. **Commit messages** (`0efccb83`, `995ed298`, `5c462bd1`):
   - The claims in `0efccb83` hold against the diff.
   - No `Pattern-Departure:` trailer is present, and none is needed, since no PATTERN entry forbids the change.
   - No PATTERN changes are declared in the directive, so nothing is owed under "Declared PATTERN changes landed".
   - Covered.

**Findings**

- **F1. LOW, consistency, `internal/sheddriver/drive.go:352`:**
  - **Summary:** the `handOver` doc says a failed spawn leaves the run parked "for the parent or a watching batten to pick up". The hand-over can now park over a dirty tree, and there batten's binary re-step refuses to act.
  - **Detail:** with the guard gone, a failed `SpawnSuccessor` over uncommitted paths leaves a parked, driverless run at a changed build. Batten's re-step checks `ChildDirtyPaths` and skips with "not re-stepping the halted child on the new build; the task pair has uncommitted paths" (`internal/battenshed/innerrun_binaryrestep.go:121-130`). So in that case only the parent picks the run up.
  - **Summary claim:** for the same reason, `_lym/landing/summary.md`'s line "this failure no longer reaches that path" overstates things for the failed-spawn case.
  - **Evidence:** `drive.go:352`, `innerrun_binaryrestep.go:121-130`, `_lym/landing/summary.md` (Design decisions, third bullet).

There are no other findings.

**Tool calls**

1. Grepped the repo and `_bolt` specs for the removed hand-back message texts, and for clean-tree hand-over wording in `pattern`, `PATTERN.md` and the stencils; read the `battencli` `dirtyPathsOutsideRunRecords` caller. The context lacked consumers of the removed strings and the other `PathsOutsideRunRecords` site.
2. Grepped for clean-tree wording in the driver, batten and loom packages and docs. The output was oversized; the context lacked doc statements beyond `loomcli/doc.go`.
3. Narrowed that grep to hand-over, driver and build lines, and listed the `pattern` files that mention a hand-over, for the same gap as call 2.
4. Read the hand-over lines in `pattern/PATTERN-shed-driver.md` and the hand-over, re-step and clean mentions in the `battenshed` and `sheddriver` doc.go files, plus `innerrun_binaryrestep.go:95-135`. These are the rule doc and batten's re-step guard, which the manifest lacked.
5. Read `internal/battenshed/stophold.go:40-90`, to see whether batten reads the hand-over record when it holds a stop. That is one hop beyond the manifest.
