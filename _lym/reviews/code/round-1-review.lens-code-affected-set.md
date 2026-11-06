```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, covering what the diff forces outside the lines it touches.

**Per change, the set found:**

1. **`shedrun.UsageDirRel()` added to `loomRecordsDirs`** (`driftwiring.go:283`)
   - The only production reader is `Deps.LoomRecordsDirs` at `driftwiring.go:244`, consumed by `isLoomRecordsPath` at `judgesnapshot.go:220`. Covered.
   - The `Deps.LoomRecordsDirs` field comment (`deps.go:60-61`) is generic: "the records paths loom itself writes during a row". It stays true. Covered.
   - The `loomRecordsDirs` doc comment is updated. Covered.
   - `driftengine/doc.go:95` is updated to name the usage records. Covered.
   - Tests that build `LoomRecordsDirs` by hand are `judgesnapshot_test.go:47` and `verify_test.go:211`. Neither lists the usage directory.
     - They don't need to. `driftwiring_test.go` pins the wiring, and the engine is path-agnostic. Not a finding.
   - `export_test.go` re-exports `loomRecordsDirs`, and the wiring test uses it. Covered.
   - No spec, stencil or `PATTERN` text describes the exempt list.
     - `PATTERN-run-records` and `PATTERN-sole-parsers` name the writers and parsers of usage records, not the judge-snapshot exemption.
     - The grep over the repo outside `_lym`/`.lym` found no other hit. Covered.

2. **`isJudgementPath`, replacing `rel == judgementRel` on both checkouts** (`judgesnapshot.go:207,220,235-246`)
   - It has two call sites, both updated.
   - The `Changes` doc comment and `doc.go` are updated. Covered.
   - The `driftengine` import of `shedadapters` is new in this file only.
     - `deps.go`, `verify.go`, `expect.go` and `verbs.go` already import it, so there is no new package edge. Covered.
   - Tests: the new `TestJudgeSnapshot` rows cover a collision suffix, a wrong extension, a different directory and a non-stamp name. Covered.
   - `driftwiring_test.go`'s file header is updated. Covered.
   - `judgement.yaml` is also rotated by `archiveStaleOutputs` (`archive.go:37-40`) as `<base>-<stamp>[-n]<ext>`.
     - The regex `^(.+)-\d{8}T\d{6}Z(?:-\d+)?$` matches that shape. The extension-trim logic is consistent with the rotation. Covered.

3. **Commit message and `summary.md` claims**
   - Both claims hold against the diff: the usage directory is exempt and archived judgements are exempt on both checkouts.
   - The summary's "Bound" statement matches the `doc.go` "Bound" line, which still reads "under the exempt loom paths", so it now also covers the usage directory and archived judgements, both loom/lym-written. No contradiction.

4. **`PATTERN` declared changes:** none declared, and `PATTERN.md` and `pattern/` are untouched. Nothing to land.

5. **`Pattern-Departure:` trailers:** none in the commit, and none needed.

**Findings:** none.

**Tool calls:**
1. `grep` across the repo for `LoomRecordsDirs`/`loomRecordsDirs` and the judgement-exemption wording, to find every reader, test and doc beyond the manifest. My context lacked `deps.go`, `verify_test.go` and any non-Go references.
