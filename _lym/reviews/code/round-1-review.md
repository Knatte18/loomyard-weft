---
verdict: APPROVED
---

The review range is `fc7f7d10a..HEAD`, two commits, all in `internal/preflight`.
The diff adds `shedrun.UsageDirRel()` to `runStateExemptRels`, updates the function's and the package's docs, and adds a session usage record to `TestCheckResolved_LymOwnStateIsExempt`.

None of the four lenses (code-correctness, code-affected-set, code-tests, code-scope) or the handler's own review found a defect.

How each focus entry was settled:

- **Rooting of `UsageDirRel`.**
  `UsageDirRel` is `filepath.Join(lymdirs.LymDirName, lymdirs.UsageDirName)` (`internal/shedrun/usage.go:29`).
  It is anchor-relative and rooted at `_lym`, the same as `RunsRootRel` (`internal/shedrun/paths.go:475`) and `LoomFrictionDirRel` (`internal/loomengine/config.go:358`).
  `fabricengine.Clean` scopes each exempt path under `AnchorRel` and adds it as a `:(exclude)` pathspec (`internal/fabricengine/warpclean.go:52-61`).
  Git applies an exclude pathspec whatever the path's status, so untracked, modified and deleted usage records are all skipped.
  The test's `backend` subtest pins the subpath anchor.
- **Scope.**
  Both issues name excluding `_lym/usage` as an accepted fix, so exempting the whole directory, `run.json` and `landed.json` included, is licensed.
  Nothing outside the usage directory is newly exempt:
  the test's `_lym/plan/plan.md` assertion still refuses.
- **Test.**
  The record goes through the real writer, `shedrun.WriteSessionUsage`, into `shedrun.UsageDir(l)`.
  The record's lock lands in the never-tracked lock directory (`internal/lock/tracked.go:29-44`), so it adds no dirty path.
  The unscoped `fabricengine.Clean` assertion proves git sees the record, and the closing assertion still pins that a file outside the exempt state refuses.
- **Doc comments.**
  The new sentences in `preflight.go` and `doc.go` describe the code.
  Later records commits do pick up the usage directory (`internal/loomcli/sharedbootstrap.go:187`, `internal/loomcli/wiring.go:130`), so the existing sentence "stay uncommitted until a later records commit" holds for the new entry too.

Commits: the `Root-Cause:` trailer on 20977989f names the omission from `runStateExemptRels`, the defect the diff removes.
Merge danger: the call is `two-way` with blast radius `preflight`.
The diff changes no record or board format and no config meaning, deletes no data, and pushes nothing, so both hold.
No PATTERN changes are declared.

## Rejected

- The `preflight.go` doc names only the session record as the usage directory's content, though `run.json` and `landed.json` are exempt too: rejected, because the sentence is true as written and the exemption's reason, lym's own bookkeeping in flight, covers every record in the directory.
