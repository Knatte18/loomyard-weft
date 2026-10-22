---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    origin: lens:code-pattern-fit
    location: internal/websterengine/recordbatch_test.go:1698
    summary: glyphRejectedWayForward's uncheckable branch (fresh-restart steps) has no test reaching it
    fix: In TestRecordBatch_DoneChecksBlockOnUnresolvedCreate add a GlyphRejected subtest on a record whose Batch.Uncheckable is non-empty (reuse the fixture the delete-referenced uncheckable test uses), asserting the message contains the fresh-restart steps and does not contain recover-batch.
  - id: F2
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/planglyph/donecheck_resolve_test.go (TestDoneChecks_Verdicts table)
    summary: No Rename row covers the disk route, though the diff changes both rename-not-done arms for onDisk entries
    fix: Add rows to TestDoneChecks_Verdicts with a Rename pair of spaced paths - old still present with new missing reports rename-not-done on both sides, old gone with new present passes.
  - id: F3
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/planparser/glyphref.go:112
    summary: IsDiskCheckedTarget and the gateDiskCheckedTarget ledger row have no direct test of the keep/skip split against glyph-spelled targets
    fix: Add a table test in internal/planparser asserting IsDiskCheckedTarget is true for a slashed path, a bare root filename and a spaced path, and false for a self glyph, a member glyph, a stripped glyph half of a plan handle and a bare symbol.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/planglyph/donecheck.go (searchedDescription, elsewhere loop)
    summary: The same-name declarations list is neither sorted nor bounded, so its order follows the listing and a common name such as New names every declaration in the worktree
    fix: Sort elsewhere before joining, as unitFiles is sorted; the unit-file list stays unbounded as the decision record states.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/planglyph/donecheck.go (diskLookup)
    summary: A stat answering ENOTDIR, such as a Delete target whose parent directory became a file, returns an error instead of reading as nothing exists there
    fix: In diskLookup treat errors.Is(err, syscall.ENOTDIR) like fs.ErrNotExist, returning false with the nothing-exists detail.
  - id: F6
    severity: NIT
    class: consistency
    origin: lens:code-pattern-fit
    location: internal/planglyph/donecheck_test.go (TestDoneCheckVerdicts_Rules table, "create missing, unit missing" row)
    summary: Two table rows share one line after the edit, breaking the one-row-per-line layout
    fix: Put the {"delete gone", ...} row on its own line.
---

### [MEDIUM] Uncheckable glyph-rejected branch is untested

**Location:** internal/websterengine/recordbatch_test.go:1698
**Issue:** `PATTERN-refusal-way-forward` requires a test to reach each way forward a refusal names. The new `GlyphRejectedNamesThePlanEdit` subtest reaches only the plain branch; the branch returning `freshRestartSteps(stepRun)` for a record with uncheckable entries is never exercised.

### [LOW] No Rename disk-route row

**Location:** internal/planglyph/donecheck_resolve_test.go
**Issue:** Create and Delete disk verdicts are pinned; the `rename-not-done-old` and `rename-not-done-new` disk arms are not, though the board entry's disk route covers them.

### [LOW] IsDiskCheckedTarget has no direct test

**Location:** internal/planparser/glyphref.go:112
**Issue:** The focus asks that the gate match exactly the path-shaped targets with no overlap with glyph-spelled ones. Only the indirect planglyph rows reach it, so a ledger regression on glyph or handle kinds would go unseen.

### [LOW] Same-name declaration list unsorted and unbounded

**Location:** internal/planglyph/donecheck.go (searchedDescription)
**Issue:** `unitFiles` is sorted but `elsewhere` follows listing order, so the detail text is order-dependent and, for a common member name, can list a large part of the worktree into a refusal message.

### [LOW] ENOTDIR on the disk route errors out

**Location:** internal/planglyph/donecheck.go (diskLookup)
**Issue:** Only `fs.ErrNotExist` reads as absent. A Delete whose parent path component is now a regular file stats as ENOTDIR and aborts the record with an error although nothing exists at the path.

### [NIT] Joined table rows

**Location:** internal/planglyph/donecheck_test.go
**Issue:** `}}, {"delete gone", ...` puts two rows on one line.

## Rejected

- Path escape via `..` or an absolute path in `diskLookup`: `checkCardPathMalformed` (`card-path-malformed`) in `internal/planparser/validate.go` refuses both at plan validation, a trust boundary upstream of this code.
- Untested `ErrQuarryUnavailable` wrap of a listing error: the wrap is a one-line mirror of the resolve error's, and no seam provokes a listing failure after a successful resolve.
- Redundancy of the `searched` assertions in `TestDoneCheckVerdicts_Rules` against `TestDoneChecks_Verdicts`: the Rules rows extend an existing table and add the Rename-new check ID.
- No test that the listing is skipped when no finding is due: it is an internal cost property, not an observable outcome.
- Glyph-rejected way forward lost when the cause is flattened (`failCardNotDone` else branch): the only flattened cause (`recordbatch.go:968`) carries bind findings, never glyph-rejected.
- Delete-referenced way forward taking precedence over the glyph-rejected one: an edge combination, and the precedence is the existing structure's order.
- `searchedDescription` allowlist entry looking unneeded: harmless, and the scan matches it by name.
- `webster validate` and the `loop-exited` halt reason: the diff touches neither, so the scope note holds.
- Summary claims: each checked claim (disk route, not-found detail, way forward, fold of the root-filename test, allowlist additions, docs) is borne out by the diff.
