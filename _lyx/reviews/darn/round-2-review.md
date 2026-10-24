---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/configsync/movekeys.go:271
    summary: a former-default source value dropped from a prime-seeded source is never reported, because the drop line lands on a target file the move does not mark changed
    fix: In takeKey, when the drop branch (formerSourceDefault true) appends to reportOn.keptTarget and reportOn is the target of a virtual source, make moveKeys' results loop, which lists only changed files, carry the line. Add a row to TestReconcile_MovesKeysAcrossModules with the darn.yaml absent from the board dir, the prime holding a former default such as review_hard_max_bounces 2, and SeedDefaults on, asserting the "dropped, a former template default" line. Setting changed would make the write loop rewrite the target, so instead add a moveFile field such as reported, set it where a report line is appended, and have the results loop select files with changed or reported while the write loop still writes only changed files.
---

### [LOW] A dropped former default from a prime-seeded source is unreported

**Location:** `internal/configsync/movekeys.go:271`

**Issue:** When the board dir has no darn.yaml and `seedDir` is set, `loadSource` reads the prime's copy and marks the file `virtual`.
`takeKey` then sets `reportOn` to the target, since a virtual source is never written.
The new drop branch appends its "dropped, a former template default" line to `reportOn.keptTarget` and returns, without setting `target.changed`.
`moveKeys` builds its results only from files with `changed` set, so the line is lost.
The same happens in the existing kept-target branch, so the gap is not new, but the drop branch adds a new silent path.
The value is still dropped correctly: nothing is moved and the target is not written.
Only the report is missing; the key-move report is how an operator sees a value that did not carry over.
A tuned value still moves (the move branch sets `changed` and `gained`), the moved-key set that exempts a target from the strip pass is built from `Moved` lines only, and a dropped value never enters it, so no key is left unmoved and unreported in the strip pass itself.

## Verdicts on the focus questions

- A darn.yaml value equal to the former template default is dropped rather than moved over a loom.yaml value, for `review_hard_max_bounces` 2, `darn_reviewer_model` `sonnet[high]` and the empty model and fan keys, whenever `overwriteDefaults` holds (a standalone or hub base, or a sparse base without the marker). A pinned (marked) sparse base and a sparse local layer move or keep the value as an explicit choice, which is the right reading.
  The current main template defaults for the other moved keys equal loom's `code_*` defaults, so their absence from `formerSourceDefaults` changes nothing.
  Tuned values (opus models, hard cap 3, amend 0) still move, pinned by the new test rows.
- `ReconcileHubWideAt` derives its exempt set from `Moved` lines, so a dropped value correctly does not enter it; the darn module's own retired keys are pruned in the same pass.
