---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    origin: lens:code-tests
    location: internal/heddleengine/settings_test.go:70
    summary: "Documented keep and take-out rules of the settings render have no row: a scalar changed since stays, a user item in a heddle-owned list stays, the record is dropped when no entry is left"
    fix: "Add rows to the table in TestRenderClaudeSettings in internal/heddleengine/settings_test.go. (1) A scalar changed since stays: file holds attribution.commit set to mine, earlier sets attribution.commit to the empty string, texts is empty with the noShipped layer, want the file unchanged. (2) A user item in a heddle-owned list stays: file holds permissions.deny with Mine and Artifact, earlier denies Artifact, texts is empty with noShipped, want deny holding only Mine. (3) No entry and no record touches nothing: noShipped, earlier nil, file holds a model key, want the file unchanged and neither written nor removed. (4) The record is dropped with the last entry: after the render that empties the entries, a third render with a new entry must not take out keys the first render set; assert it by adding the key back to the file by hand before the third render and expecting it kept. (5) A blank file and a malformed record: file holds only whitespace and wants the entries written; a record file overwritten with a JSON list before the checked render wants stale keys kept and no error. Extend the table struct with an optional record overwrite field if row (5) needs it."
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/heddleengine/settings.go:83
    summary: "An entry whose text holds no JSON object still counts as held, so a render with only such entries strips every key heddle set last time and records an empty object"
    fix: "In foldObjects in internal/heddleengine/settings.go, report failure to decode through a second result, such as invalid, and in renderObjectTarget skip the dropStale call and merge the previous record into the record it writes whenever any entry naming the target failed to decode. The invalid entry then leaves its earlier keys in the file and the record, and the warning already naming the way forward stays. Add a row to TestRenderClaudeSettings that hand-writes an invalid entry text into the store file after a good render and expects the earlier keys kept."
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/heddleengine/settings.go:265
    summary: "The record is removed before the file is written or removed in the no-entry case, so a failed write leaves stale keys in the file with no record to take them out later"
    fix: "In renderObjectTarget in internal/heddleengine/settings.go, move the os.Remove of recordPath in the not-held branch after the file's removal or writeRendered call has succeeded, and return the removal error then. In the held branch keep the file-then-record order."
  - id: F4
    severity: LOW
    class: scope
    origin: lens:code-affected-set
    location: internal/configreg/configreg.go:156
    summary: "The fabric reinterpretation heddle-claude-settings-excluded has no reconcile-level test showing a hub with its own warp_exclude gains the path once"
    fix: "Add a case to internal/hubreconcile/reinterpret_integration_test.go, following its existing DirBaseReinterpretation case. Seed a hub base fabric.yaml with warp_exclude set to A.md, run the reconcile, and expect the file to list A.md and .claude/settings.local.json, the marker for heddleengine.SettingsExcludeReinterpretation to exist, and a second run to change nothing."
---

### [MEDIUM] Keep and take-out rules of the settings render are untested

**Location:** internal/heddleengine/settings_test.go:70
**Issue:** The focus asks whether keys heddle never set are kept, including after a Claude-written permission, and the change description claims that a scalar the file changed since stays.
`TestRenderClaudeSettings` pins the main take-out row and a Claude-written `allow` list, but not a scalar changed since, a user item inside a list heddle also owns, the record being dropped when the last entry goes, a blank file or a corrupt record.
Each is behavior `dropStale` and `renderObjectTarget` add, and a regression in any would pass every test.

### [LOW] An all-invalid entry set strips the keys heddle set last render

**Location:** internal/heddleengine/settings.go:83
**Issue:** `foldObjects` sets `held` before it decodes an entry, so a hand-edited entry or override that holds no JSON object leaves `held` true and `folded` empty.
`renderObjectTarget` then calls `dropStale` against that empty fold and overwrites the record with an empty object.
The warning says the entry is left out, yet the keys it set earlier, such as the deny list, are removed from the file.

### [LOW] Record removal precedes the file change in the no-entry case

**Location:** internal/heddleengine/settings.go:265
**Issue:** With no entry left, `renderObjectTarget` removes the record before it writes or removes the file.
When the write or removal then fails, the file keeps keys heddle set and no record names them, so no later render takes them out.

### [LOW] The settings reinterpretation is wired without a reconcile test

**Location:** internal/configreg/configreg.go:156
**Issue:** `TestAppendExcludePaths` tests the rewrite over bytes.
Nothing shows the registry row, the reinterpretation ID and the apply-once marker take a hub's own `warp_exclude` through the real reconcile, which the decision record asks for.
The tests-lens report rated this MEDIUM and the affected-set lens LOW; the rewrite itself is covered and the registry row is a single line, so it is kept at LOW.

## Focus verdicts

- Fold and merge in `settings.go` follow the record: objects merge per key at every depth, a later scalar replaces an earlier one, lists join without repeats, and the heddle scalar overwrites the file's.
- Removal of what heddle wrote last and no entry sets now is correct for keys, list items and emptied containers, and keeps keys heddle never set, including a Claude-written `allow` list, a missing file and a malformed file; only the gaps in F1, F2 and F3 remain.
- Entries that are no JSON object are refused at `Set`, `SetOverride`, `ValidateEntry` and the loaders, and the markdown targets render as before.
- `shipped.json` carries the removed file's deny list in the same order, `ly@loomyard` on with no marketplace, `scribe@scribe` on with the scribe marketplace from GitHub `Knatte18/scribe`, and an empty attribution; nothing of `mill@millhouse`.
- `.claude/settings.local.json` is in the template's `warp_exclude`, fabric seeds the exclude lines before the seam fires, and a hub with its own list gains the path through the new reinterpretation.
- The seam's made event renders every target, so the prime at `fabric clone` and an added worktree both get the file.
- The record's question on targets declaring their paths is settled in the diff and `PATTERN-heddle-instructions.md`: fabric cannot read heddle, so a new target keeps a second edit in fabric's template; this is consistent with `PATTERN-fabric-boundary`.
- `PATTERN-heddle-instructions.md`, `docs/overview.md` and both `doc.go` changes land with the code, and the commit messages carry no `Pattern-Departure:` trailer and need none.

## Rejected

- Tests lens: `AppendExcludePaths` has no row for a non-mapping document, a non-scalar `warp_exclude` or several paths in one call; the first two are defensive branches for input the reconcile does not produce, and the variadic form is called with one path.
- Correctness lens: an entry-set empty container such as an empty `env` object is never removed; cosmetic, with no setting left behind.
- Correctness lens: the YAML re-encode in `AppendExcludePaths` reindents the whole file; formatting only, and comments and keys are kept.
- Tests lens: `TestShippedClaudeSettings` overlaps `ShippedEntriesArePinned`; the first pins the content and the second the slugs and targets, so neither duplicates the other.
