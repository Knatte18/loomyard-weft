# Preflight worktree-clean skips lym's own `_lym/usage` records

Preflight's worktree-clean check now ignores uncommitted files under `_lym/usage`.
That directory holds lym's per-session usage records, written when each agent session ends.
A session can end after the run's records commit and before the check.
When that happened, the check reported the untracked `session-<id>.json` as a dirty `_lym` change and halted the run as `blocked`, though lym had written the file itself (issues #190 and #194).

## Change

`runStateExemptRels` in `internal/preflight/preflight.go` names the paths under `_lym` that worktree-clean skips.
It now lists `shedrun.UsageDirRel()` beside the run-records root and the friction directory.
The records commits that already sweep `_lym` still commit the usage records.
Preflight just no longer waits for them.
Every other path under `_lym` still counts as dirty.

## Design decisions

- Exempting the directory in Preflight was chosen over committing the usage records before the check.
  A commit there would still race a session that ends after it.
  Preflight already keeps one exemption list for lym bookkeeping that is in flight, and the usage records belong on it.
- The whole usage directory is exempt, not only `session-*.json`.
  Its other records, `run.json` and `landed.json`, are also lym's own writes.

## Tests

`TestCheckResolved_LymOwnStateIsExempt` now writes a session usage record through `shedrun.WriteSessionUsage` alongside the other exempt state.
It asserts that worktree-clean still passes.
On the merge base, the record falls outside the exempt paths, so the test's clean assertion meets a worktree-clean failure there.

## Merge danger

**Door:** two-way
**Blast radius:** preflight
