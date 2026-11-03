# Preflight: stop refusing lym's own friction notes under _lym

Preflight's worktree-clean check halted a run as `blocked` over friction notes lym itself had written or moved under `_lym`.
In the reported case the previous halt's `_lym/loom/friction/loom-halt.md` was deleted, because a reflection had moved it into its archive, and that uncommitted deletion caused the next halt.
Each halt's note could cause the next one, and only a hand `lym fabric commit` let the run continue.

Closes #33

## What changes

- `runStateExemptRels` in `internal/preflight/preflight.go` adds the loom friction directory (`loomengine.LoomFrictionDirRel()`) and its reflection archives to the state-side exemptions, beside the run-records root.
  A new note, a note's deletion and an archive's untracked content no longer fail the check.
  Everything else under `_lym`, and the whole code side, still counts.
- `loomengine.LoomFrictionArchivesRelPattern()` is new: the anchor-relative git pathspec glob `_lym/loom/friction-*` that matches every timestamped archive sibling, so the `friction` segment stays named only in `loomengine`.
- `fabricengine.Clean`'s doc states that an exempt entry may be a pathspec glob; the code already passed entries through unescaped.
- `TestCheckResolved_RunStateUnderLymIsExempt` becomes `TestCheckResolved_LymOwnStateIsExempt`.
  It commits a halt note, removes it, writes it into an archive and adds an untracked note, at a root and a subpath anchor, and asserts the check still refuses a file elsewhere under `_lym`.

## Design decisions

- The settle-log half of the issue was already fixed by the run-records-root exemption (#27), so this change covers only the friction notes.
- The check ignores the friction state rather than committing it at each write or move.
  That matches how the run-records root is handled, and it covers the agent-written notes and the reflection's archive move, which happen outside any step that commits.
  The status commits still stage the loom durable directory, so the notes stay durable.
- Only the friction directory and its archives are exempt, not the whole `_lym/loom` directory, which also holds the plan amendment record and the rework rounds.
- `internal/preflight` now imports `internal/loomengine` to name the paths, since `loomengine` is their sole declarer; the import adds no cycle.
