---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/driftengine/judgesnapshot.go:207
    summary: the archived-judgement exemption on the code checkout has no test row
    fix: In internal/driftengine/judgesnapshot_test.go, add a TestJudgeSnapshot row beside ArchivedJudgementIsNoChange that writes an archived judgement name such as _lym/drift/judgement-20261106T083132Z.yaml under f.code.Root, lists it in f.code.Untracked, and expects no changes.
  - id: F2
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/driftengine/judgesnapshot.go:245
    summary: no test row fails if the archived name's base is no longer compared with the judgement's own base
    fix: "In internal/driftengine/judgesnapshot_test.go, add _lym/drift/other-20261106T083132Z.yaml to the others list of the OtherFileBesideTheJudgementIsNamed row and the string \"records: _lym/drift/other-20261106T083132Z.yaml was added\" to its expected changes, in sorted position."
---

### [LOW] Code-side archived-judgement exemption is untested

**Location:** internal/driftengine/judgesnapshot.go:207
**Issue:** `Changes` now applies `isJudgementPath` to the code checkout as well as the records checkout. Every new `TestJudgeSnapshot` row puts the archived file on `f.records`. Reverting the code side to `rel == judgementRel` fails no test. The behavior is the diff's own addition, so it needs its own row.

### [LOW] Base-name comparison in `isJudgementPath` is not pinned

**Location:** internal/driftengine/judgesnapshot.go:245
**Issue:** The `archivedFrom == ...` comparison is what keeps an unrelated stamped file from being exempt. The negative rows fail earlier: `judgement-notes.yaml` has no stamp, and the other two have the wrong extension or directory. Dropping the comparison, so any `<name>-<stamp>.yaml` beside the judgement is exempt, keeps every test green. This is the widening the focus directive asks about, and no row guards it.

## Rejected

- Focus question on the code-checkout exemption not being licensed by the entry: the entry asks that the snapshot check exempt lym's own records under `_lym/drift/`, and `judgement.yaml` was already exempt on both checkouts, so applying one predicate to both keeps them in step. Not a defect.
- Focus question on `isLoomRecordsPath` matching a bare string prefix: the usage entry has no trailing `-`, so it needs an exact match or an `entry + "/"` prefix, and `_lym/usagefoo` is not exempt. Not a defect.
- Missing test for the usage directory as a prefix rather than a string prefix: the function is unchanged by the diff and the new wiring test pins the path itself. Not raised.
- `export_test.go` addition: needed, because the wiring test is in the external test package and `loomRecordsDirs` is unexported.
- Other judge-written paths that still halt: usage `run.json` and `landed.json` sit in the same directory and are now exempt, locks are placed outside the tracked directory, and I found no other lym-written path under the records checkout. Nothing beyond the entry is widened.
- Doc comments and commit message: `doc.go`, the `Changes` comment, the `loomRecordsDirs` comment and the commit message match the diff. No `Pattern-Departure:` trailer is present and none is needed. No PATTERN entry is declared changed.
