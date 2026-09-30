---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    location: internal/fabricengine/mergestate.go:260
    summary: foreignMergeStatePresent's doc comment did not move to the extracted reader, so the unconditional-evaluation rationale is stated twice and the probe-kind paragraphs sit on the function that no longer runs the probes
  - id: F2
    severity: MEDIUM
    location: internal/fabricengine/midmerge.go:1
    summary: new and changed Go comments are wrapped at a fixed column mid-sentence, or put two sentences on one line, contrary to CONSTRAINTS.md's Comment Line-Break Convention
  - id: F3
    severity: LOW
    location: internal/loomcli/start_midmerge_test.go:122
    summary: the spawn-refusal table and the two probe-error rows never assert that the driver sender was not called, which card 3 requires of every refusal row
  - id: F4
    severity: LOW
    location: internal/loomcli/start_midmerge_test.go:136
    summary: dirOf hand-rolls filepath.Dir, and panics on a path without a separator
  - id: F5
    severity: NIT
    location: internal/fabricengine/midmerge.go:62
    summary: MidMerge returns readForeignProbes's error bare, while every other failure path carries the mid-merge probe prefix
  - id: F6
    severity: NIT
    location: internal/loomcli/start_midmerge_test.go:2
    summary: the file-header comment abuts the package clause, so it becomes a package doc comment, unlike every other file's header in the package
---

Review range: `product.parent` is `main` (from `_lyx/shed/loom-start-refuses-mid-merge/status.json`), so the range is `git diff $(git merge-base main HEAD)..HEAD`, three commits matching cards 1–3.

Evidence gathered: `go build ./...`, `go vet` (untagged and `-tags smoke`) over the touched packages, `go test` over `loomcli`, `fabricengine`, `shedrun` and `battencli`, and `go test -tags integration -run 'MidMerge|ForeignMergeState|MergeVerbs' ./internal/fabricengine/` all pass.

### [MEDIUM] foreignMergeStatePresent's doc comment did not move with the four calls

**Location:** internal/fabricengine/mergestate.go:260 (readForeignProbes) and :281 (foreignMergeStatePresent)
**Issue:** Card 2 says foreignMergeStatePresent's existing doc comment "moves with the four calls where it describes them".
The diff instead writes a fresh comment on readForeignProbes restating the unconditional-evaluation rationale, and leaves foreignMergeStatePresent's comment untouched.
The rationale now appears twice, and on foreignMergeStatePresent it is false: that function no longer evaluates anything, it combines readings.
The paragraphs on the two probe kinds, the per-probe test pinning and the kept weft probes also describe the four calls, and now sit one function away from them.
The answer and error wrapping of foreignMergeStatePresent are byte-identical for its six callers (same four `fmt.Errorf` strings, same `||` combination), so behaviour is not at issue.
**Fix:** Move the unconditional-evaluation sentence and the three following paragraphs onto readForeignProbes, and cut foreignMergeStatePresent's comment down to what it answers plus a pointer to the reader.

### [MEDIUM] Comment line-break convention violated in new and changed comments

**Location:** internal/fabricengine/midmerge.go:1, :19, :22, :27, :37–:49; internal/fabricengine/mergestate.go:250, :259; internal/fabricengine/doc.go:510–:513; internal/loomcli/cli.go:83; internal/loomcli/start.go:124, :165; internal/loomcli/cli_test.go:162, :186; internal/loomcli/start_midmerge_test.go:1
**Issue:** CONSTRAINTS.md's Comment Line-Break Convention requires one sentence per line, broken further only at an independent-clause boundary, with no column limit, for every new or changed Go comment.
These comments are instead wrapped at roughly column 100, mid-phrase (for example `a fabric merge verb stopped` / `part-way`, `the same form a fabric` / `merge verb's conflicts use`, `a conflicted` / `warp-side merge-in`).
Several also put two sentences on one line: readForeignProbes (`carries the state. It is the one place`), the midMerge field (`driver. The Test Tier Purity Invariant`), and the test file header (`alone. Fakes only, no git.`).
In start.go:165 the break before `but only when` splits a dependent clause, not an independent one.
The card-1 comment in shedrun/paths.go complies.
**Fix:** Rewrap each listed comment to one sentence per line, breaking inside a sentence only at an independent-clause boundary.
For cli_test.go:186 the changed line is inside one sentence, so that whole sentence is rewrapped; untouched sentences around it stay as they are.

### [LOW] Refusal rows do not assert the sender was never called

**Location:** internal/loomcli/start_midmerge_test.go:122, :225, :247
**Issue:** Card 3 requires each refusal row to assert "that the starter and sender were never called".
Only TestRunDriverSpawnAndWait_MidMerge_ParkedLiveDriverRefused checks the sender.
The spawn table, the probe-error row and the real-probe row build their receiver through newTestSpawnAndWaitReceiver, which leaves driverSender nil, and assert only the starter.
The probe-error and real-probe rows also skip the driver-log assertion the card names for every refusal row.
The go-arm row itself is sound: an unrefused go arm would either spawn or fail with a kindless envelope, and the `kind` assertion rules out both.
**Fix:** Give those receivers a `fakeDriverSender` and assert it received no text, and add the driver-log assertion to the two probe-error rows.

### [LOW] dirOf duplicates filepath.Dir

**Location:** internal/loomcli/start_midmerge_test.go:136
**Issue:** dirOf reimplements `filepath.Dir`, which newResumeBranchReceiver in the same package already uses for the same marker path, and panics with an index-out-of-range on a path without a separator.
**Fix:** Delete dirOf and call `filepath.Dir(marker)`.

### [NIT] MidMerge returns the foreign-probe error unwrapped

**Location:** internal/fabricengine/midmerge.go:62
**Issue:** Open, mergeRecordExists and the geometry and unmappable failures all carry `fabricengine: mid-merge probe: ...`, but a readForeignProbes failure reaches start's envelope as a bare `fabricengine: check merge head: ...`.
It is still an error, so the probe-answers-never-default-to-clean decision holds; only the operator loses the hint that start's mid-merge check produced it.
**Fix:** Wrap it as `fabricengine: mid-merge probe: read git merge state: %w`.

### [NIT] Test file header becomes a package doc comment

**Location:** internal/loomcli/start_midmerge_test.go:1–3
**Issue:** The header comment sits directly above `package loomcli` with no blank line, which makes it a package doc comment.
Every other file header in the package, and midmerge.go's own, is separated from the package clause by a blank line.
**Fix:** Insert a blank line between the header and the package clause.

## Focus directives settled without a finding

- start.go's switch sends every kind other than None and Parked to the foreign message.
  That fails closed (it refuses) for an unrecognised value, and MidMergeKind has exactly three values, so it is sound.
- Hoisting the park-marker stat leaves every path unchanged.
  When mustSpawn is true the stat result is unused except for the probe gate.
  When mustSpawn is false and the strand is live, `parkedLive` equals the old inline `statErr == nil`, so the resume, stale-marker removal and `driver_not_parked` paths behave exactly as before.
- Fabric Vocabulary Invariant: no identifier, comment or envelope message added to internal/loomcli names either side.
  MidMergeKind, its constants, MidMergeState and their doc comments are side-neutral.
  MidMerge's own doc comment and its unmappable error name the sides, as card 2 itself requires for the doc comment; both live in fabricengine, an owner-set package.
- Per-card mechanical checks: `SetsBothInjectedSeams` has no remaining reference outside `_lyx`.
  foreignMergeStatePresent's six call sites see an unchanged answer.
  Both constructors go through newLoomCLI, which sets midMerge.
  newTestSpawnAndWaitReceiver's consumers get a MidMergeNone fake, and the untagged suite passes.
  Nothing equivalent already existed: MergeStateActive is weft-only, MergeInProgress is record-only, and no existing start refusal kind covers a merge.
