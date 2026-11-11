---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/boardengine/replay.go:190
    origin: lens:code-affected-set
    summary: replayFields has no record_body_sha256 row, so a board sync replay does not carry the stamp although D5 and doc.go say it does
    fix: In replay.go add a replayFields row named record_body_sha256 beside the discussed row, with Get returning t.RecordBodySHA256 and Set copying src.RecordBodySHA256 into dst. Add a replay test in the boardengine replay tests where a local write changes body and stamp and the replay onto origin's entry carries both.
  - id: F2
    severity: MEDIUM
    class: scope
    location: internal/boardengine/recordupsert_test.go:540
    origin: lens:code-tests
    summary: The family-project exemption is pinned only for the all-true case, and no row shows it never exempts a non-filed project or a run-held family project
    fix: In recordupsert_test.go add rows to TestUpsertTaskWithRecord_Refusals, or beside the family-project test, for a kind project entry the raddle store does not record as filed (worktree standing refused, live weft refused), for a filed part where Holds is true but Project differs from the slug, and for a run-held family project refused with the run-held way forward.
  - id: F3
    severity: LOW
    class: scope
    location: internal/boardengine/store.go:1183
    origin: lens:code-affected-set
    summary: The merge path keeps the discussed flag on a removed and re-upserted entry but drops record_body_sha256, so promote then gives no warning for a changed body
    fix: In the keepDiscussed branch at store.go:1193 also carry removedEntry.RecordBodySHA256 onto s.tasks[i] and onto upserted. Add a merge test row asserting the stamp survives.
  - id: F4
    severity: LOW
    class: consistency
    location: plugins/ly/skills/board/SKILL.md:94
    origin: lens:code-affected-set
    summary: The board skill documents upsert --body-file and promote but not upsert --record-file or promote's new warning key
    fix: In plugins/ly/skills/board/SKILL.md add a short paragraph for upsert --record-file, naming its refusals and that a run's task session cannot use it, and a note that promote of a Sketch adds a warning when its body changed after the record was stored.
  - id: F5
    severity: LOW
    class: design
    location: internal/boardengine/recordupsert.go:36
    origin: lens:code-correctness
    summary: Every probe error is wrapped with the way forward to re-run once origin answers, which is wrong for the --board-path and config-load errors from the CLI probe, and a missing test leaves that path unreached
    fix: In recordupsert.go make the probe-failure way forward neutral, for example re-run once the probe's cause is resolved, or return a distinct error from the cli.go recordsBranchLive closure for the location-nil and config cases. Add a TestCLI row for upsert --record-file under --board-path expecting that refusal and nothing written.
  - id: F6
    severity: LOW
    class: design
    location: internal/boardcli/recordfile.go:167
    origin: lens:code-correctness
    summary: --record-file does not read stdin for the value - although the flag is meant to mirror --body-file, so a file named - is read instead
    fix: Either accept stdinPath in readRecordFile and read stdin through the cobra command's input, or state in the flag's help text and in doc.go that stdin is not read.
  - id: F7
    severity: LOW
    class: scope
    location: internal/fabricengine/add_remoteleftover_integration_test.go:828
    origin: lens:code-tests
    summary: The local-branch row of TestRecordsBranchLive has a reachable origin, so it would pass if the local branch did not decide alone
    fix: Add a row with a local weft branch and an origin set to a missing URL, expecting live true and no error.
  - id: F8
    severity: LOW
    class: scope
    location: internal/boardengine/sketch_test.go:617
    origin: lens:code-tests
    summary: No promote-warning row shows a stamped entry that is not a Sketch, with an edited body, gets no warning
    fix: In TestPromoteToRecordWarning add a row for a stamped task or note without the sketch label and with an edited body, expecting an empty RecordWarning.
  - id: F9
    severity: LOW
    class: consistency
    location: internal/boardengine/recordupsert_test.go:540
    origin: lens:code-tests
    summary: TestUpsertTaskWithRecord_FamilyProjectSkipsWorktreeAndWeftRefusals is listed as contained but pins the exemption no covering test reaches, and carries no testtiming keep marker
    fix: Keep the test and add a testtiming keep directive directly above its func Test line, with the reason that it pins the family-project exemption from the worktree and live-weft refusals, which TestFileRaddle and the other two upsert tests never reach.
  - id: F10
    severity: LOW
    class: design
    location: internal/boardcli/cli.go:80
    origin: handler
    summary: The CLI probe takes its skip from the fabric skip environment, but the real Add callers pass empty AddOptions, so under that environment the refusal and a raddle start can disagree
    fix: In cli.go keep the environment skip only if D4's same-as-Add guarantee is limited to the test and CI bypass, and state that in the closure comment and in boardcli/doc.go. Otherwise pass an empty AddOptions as the Add callers do.
---

### [MEDIUM] Replay does not carry record_body_sha256

**Location:** internal/boardengine/replay.go:190
**Issue:** `replayFields` lists `discussed` but no stamp row.
The field is now known on `Task`, so a replay merges it by neither route, and origin keeps the old stamp against a new body.
D5 says the replay carries the stamp with the entry, and the new `doc.go` text repeats it.
A promote on another machine then warns falsely or not at all.
Confirmed by reading the table at replay.go:190-217.

### [MEDIUM] Family-project exemption pinned only for the exempt case

**Location:** internal/boardengine/recordupsert_test.go:540
**Issue:** The exemption condition is `task.Kind == KindProject && held && filed.Project == slug`.
Only the all-true case is tested, so exempting every project would pass every current test.
D4 also requires the run lock to still apply to a family project, and no row has a run-held one.

### [LOW] Merge drops the stamp

**Location:** internal/boardengine/store.go:1183
**Issue:** `keepDiscussed` keeps the flag across a removed and re-upserted entry but not `record_body_sha256`, so the entry has a record and no stamp, and promote never warns for it.

### [LOW] Board skill not updated

**Location:** plugins/ly/skills/board/SKILL.md:94
**Issue:** The skill that describes the board verbs to agents has no `--record-file` or promote `warning` text, and the repository's rule is that docs land in the same diff.

### [LOW] Probe-failure way forward misleads for non-origin errors

**Location:** internal/boardengine/recordupsert.go:36
**Issue:** A nil location (`--board-path`) or a config-load failure surfaces from the probe closure and is wrapped as "re-run once origin answers", which cannot help.
No test reaches the `location == nil` refusal.

### [LOW] --record-file has no stdin form

**Location:** internal/boardcli/recordfile.go:167
**Issue:** `--body-file -` reads stdin, and `--record-file -` reads a file named `-`.
D4 asks only for a flag parallel to `--body-file`, so the gap is small.

### [LOW] Local-branch probe row would pass with an origin read

**Location:** internal/fabricengine/add_remoteleftover_integration_test.go:828
**Issue:** The row's origin is reachable, so it does not pin that a local branch decides with no origin read.

### [LOW] No non-Sketch promote-warning row

**Location:** internal/boardengine/sketch_test.go:617
**Issue:** The `IsSketch` gate in `PromoteTo` has no row showing a stamped non-Sketch with an edited body gets no warning.

### [LOW] Contained test without a keep marker

**Location:** internal/boardengine/recordupsert_test.go:540
**Issue:** I compared its assertions with the three covering tests the report names.
`TestFileRaddle` never calls `UpsertTaskWithRecord`, and the other two use non-project entries, so none asserts the exemption.
It pins what coverage cannot see, so it stays and is marked.

### [LOW] Skip environment can differ from a real Add start

**Location:** internal/boardcli/cli.go:80
**Issue:** `Add`'s callers (`adoptcli`, `battencli`) pass no skip and `Add` reads no environment, so under `FABRIC_SKIP_GIT` or `FABRIC_SKIP_PUSH` the upsert skips the origin probe where a start would make it.
D4 asks for the same test as `Add` under `Add`'s own conditions.
Production is unaffected, since the environment is a test and CI bypass.

## Rejected

- Discarded error of the restore write in `UpsertTaskWithRecord` (lens:code-correctness note): the restore is best effort on an already failing path, and `FileRaddle` does the same.
- A failure after the save reports an error with the write saved (lens:code-correctness note): matches `boardCriticalSection`'s documented `recordErr` behavior.
- Focus questions on refusal order, the record file before board.json, the stamp on all three writers, the promote warning's compare-before-write and the shared probe under Add's conditions were checked against the diff and hold.
