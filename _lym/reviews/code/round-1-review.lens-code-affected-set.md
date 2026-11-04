```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens. For each change, I looked for what reads it or describes it outside the touched lines.

**Per-change list**

1. **Yield returns before `SaveEnvelope` (`internal/sheddriver/drive.go:204-210`).**
   - Set found:
     - `stop.EnvelopePath` is read only after the save, so a yield never reaches it.
     - `resumed = false` and the `d.rules` reset are skipped on a yield, but nothing in the yield exit reads them.
     - `internal/sheddriver/doc.go:14` and `:18` and `internal/loomcli/doc.go:257` are updated.
     - `internal/loomcli/drivewiring.go:93` `driveEnvelopePath` and `internal/sheddriver/reportfiles.go:20` `EnvelopeFileName` only write.
     - The tackler and arbiter stencils, `contracts/stencils/shed/shed-template-tackler.md` and `shed-template-driver.md`, point at `envelope_path` only for stops that were decided, never a yield.
     - `drive_test.go` covers the yield row with `wantNoEnvelope`.
   - Covered: yes.
2. **`DriverRecordsRel` replaces the three inlined checks (`internal/loomcli/wiring.go:110`).**
   - Set found:
     - The `wiring.go:88` doc comment is updated.
     - `shedrun/doc.go:36` and the `paths.go` header are updated.
     - The remaining caller is `battencli/commitstatus.go:85`.
     - The loom pathspec test (`loomcli/wiring_commitstatus_test.go`) already asserts order and emptiness.
   - Covered: yes.
3. **`battenRunCommitPaths` now stages the driver directories (`internal/battencli/commitstatus.go:85`).**
   - Set found: the `commitstatus_test.go` row covers it.
   - Covered: yes.
4. **Batten wires `CommitRecords` and commits in `Park` (`internal/battencli/drive.go:276-280` and `:327`).**
   - Set found:
     - `driverdeps_test.go` moves `CommitRecords` to the wired set.
     - `battencli/run_test.go` stubs `recordsDeps` through the new field in `cli.go`.
     - `internal/sheddriver/doc.go:67` lists "no `CommitRecords`: a done stop commits nothing" as an optional unwired seam. That stays true as a general statement.
     - The `driverDepsFor` doc comment is updated.
   - Covered: yes, except finding F-A below.

**Findings**

- **F-A, LOW, consistency:** `internal/sheddriver/doc.go:57-67`, with `internal/sheddriver/drive.go:85-86` (the `CommitRecords` comment).
  - The doc says the optional seams are unwired only to remove rows. It names only loom's and darn's `CommitRecords` as the done-stop commit, and the `drive.go:85` comment says `CommitRecords` commits "at a done stop".
  - Batten now also commits at a hand-back through its `Park` seam. That is batten-specific, and the `drive.go:231` doc comment in battencli states it. So nothing is wrong, and I found no stale text.
  - I am withdrawing this item: no finding. I list it only because I checked it.
- **No findings from the PATTERN set.**
  - I read the PATTERN.md entries from the explore step. `PATTERN-fabric-git`: batten commits through `battenCommitStatusDeps`, the same fabric path as before, with no new git call.
  - `PATTERN-durable-vs-ephemeral-state`: the drive-reports, tackler and arbiter directories are staged under `_lym`, as that entry asks.
  - `PATTERN-named-constants`: no new literal path. The commit message `"batten: run records for "` is a new literal that appears once in `commitRunRecords`, so it does not repeat.
  - `PATTERN-sole-parsers` and `PATTERN-shed-driver` are not touched.
  - No `Pattern-Departure:` trailer is present, and none is needed. No declared PATTERN change exists, so nothing is owed there.
- **Stencils:** the diff changes none.

**Tool calls (2 of 6)**
1. `grep` over md, go and yaml files for `CommitRecords` and envelope wording. My context lacked the full set of docs and stencils that mention the stop envelope or the record commit.
2. `sed` of `internal/sheddriver/doc.go:55-80` plus a `grep` of battencli files. My context lacked the doc's optional-seam list. `internal/battencli/doc.go` does not exist (ugrep warned), so I read `battencli/drive.go` only.
