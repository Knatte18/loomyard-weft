---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/state/keep.go:121
    summary: Update keeps unknown keys only on list elements whose index is unchanged, so entries that Resume or the retention sweep merely shift lose their unknown keys
    fix: In keepUnknownInList, pair each new element with an old element that encodesAs it, searching the old elements not yet paired and preferring the same index, instead of comparing only the old element at the same index. Add a row to the pauserecord Update test where the seed holds two live entries with an unknown key on the second, mutate removes the first, and the second's unknown key is still in the file. Add the same case for an expired resumed entry dropped ahead of a kept one.
  - id: F2
    severity: LOW
    class: consistency
    location: internal/pauserecord/leaf_enforcement_test.go:13
    summary: pauserecord no longer imports internal/lock in any production file, yet the allowlist, doc.go and PATTERN-leaf-packages still admit and name it
    fix: Drop internal/lock from the allowlist and from the header comment in leaf_enforcement_test.go, switch the call to scankit.AssertImportAllowlistNoStale, and change the pauserecord sentence in doc.go:30 and pattern/PATTERN-leaf-packages.md:53 to "the standard library and internal/state, and gofrs/flock through it".
---

### [MEDIUM] Unknown keys of a shifted entry are dropped

**Location:** internal/state/keep.go:121, reached from internal/pauserecord/record.go `Update`.
**Issue:** `Update` now rewrites through `state.UpdateJSON`, and `keepUnknownInList` merges an old element into a new one only when the old element at the same index encodes to the same JSON.
The callers of `Update` change list membership: `pauseengine.Resume` rebuilds `record.Live` without the lifted entries, and `withoutExpired` drops resumed entries from the front of `record.Resumed`.
Every untouched entry behind a removed one moves to a lower index, no longer matches the old element at that index, and is written without its undeclared keys.
So an older binary running `lyx resume` or any `Update` after a retention expiry silently strips the future keys of entries it never edited, which is the loss the fix set out to prevent.
The new test appends to the end of `Live`, which keeps every existing index and so does not reach this path.

### [LOW] Stale `internal/lock` in the pauserecord import contract

**Location:** internal/pauserecord/leaf_enforcement_test.go:13, internal/pauserecord/doc.go:30, pattern/PATTERN-leaf-packages.md:53.
**Issue:** The fix removed the only `internal/lock` import from the package, so its production files import the standard library and `internal/state` only.
The allowlist test uses `AssertImportAllowlist`, which does not flag an unused entry, so the three statements of the contract still admit `internal/lock` and nothing fails.
`internal/state` imports `internal/lock` and `internal/fsx` and neither imports `pauserecord`, so there is no cycle; the defect is only that the contract is wider than the code.

## Checked, no finding

- `Update` takes the write lock at `path+".lock"` through `state.UpdateJSON`, applies `withoutExpired` inside it, returns the mutated record, and writes nothing on a mutate error or an undecodable file; the callers in `internal/pauseengine` and `internal/orchengine/pause.go` see the same lock and expiry behavior.
- `writeMarker` still writes the marker whole through a temp file and rename; `writeAtomically` moved to `hold.go` unchanged and has `writeMarker` as its only caller.
- Top-level unknown keys survive `Update`, covered by the new test row.
