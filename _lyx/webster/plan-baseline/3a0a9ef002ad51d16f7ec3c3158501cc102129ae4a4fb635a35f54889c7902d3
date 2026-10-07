# Card 9 — Lock directory and retired spellings reached through fabric

**Edit:**
- `internal/fabricengine/recordsgit.go`
- `internal/fabricengine/refscanner_test.go`
- `internal/websterengine/audit_test.go`
- `internal/hubgeom/doc.go`
- `internal/hubgeom/parent_test.go`
- `internal/hubgeom/geometry_integration_test.go`

**Intent:**
D5, data part: the spellings that are data fabric must keep, the `.weft` on-disk lock directory name and the retired per-side CLI spellings, stay spelled only inside fabric's owner set, with behaviour and on-disk state unchanged.

- `recordsgit.go` gains one exported, side-neutral helper that returns the lock directory path inside a given records worktree, built from the existing unexported lock-directory name constant, and `ensureWeftLockDirAt` builds its path through it.
  The on-disk name `.weft` stays.
- `internal/hubgeom/parent_test.go` and `geometry_integration_test.go` reach the lock directory through that helper and still assert it exists after an add and is missing where they remove it; the error-text assertion matches the helper's base name instead of a literal, and the comments say "lock directory".
- `internal/hubgeom/doc.go` says the records sibling's lock directory, without the literal name.
- The retired spellings `lyx weft sync`, `lyx warp checkout` and `lyx.exe weft push` stay classified as fabric references by `RefScanner`: their `TestRefScannerMatches` rows move from `websterengine/audit_test.go` into `fabricengine/refscanner_test.go`'s match table, joining the rows already there and adding the missing `lyx.exe weft push`.
  `TestCheckFork`'s "retired per-side sync spelling" case in `audit_test.go` uses a current spelling such as `lyx fabric sync` already covered by the case above it, so it is dropped, keeping the hard-error classification covered by that case.

The hubgeom tests are this card's coverage: they fail if the helper does not name the directory fabric creates.

**ImpactSummary:** one new exported fabricengine path helper; hubgeom and websterengine change only in tests and a doc comment, and `RefScanner` itself is untouched.
