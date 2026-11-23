# Preflight worktree-clean ignores lym's own usage records

Preflight's `worktree-clean` check no longer counts `_lym/usage` as an uncommitted state change.
lym writes a usage record there as each agent session ends,
and a session can end between a run's records commit and the check.
The check then refused lym's own bookkeeping, halted the run at its first step as `blocked`,
and escalated a state lym created to the orch.

`runStateExemptRels` in `internal/preflight` now lists `shedrun.UsageDir`'s relative path
beside the run-records root and the loom friction paths.
Its doc comment and the package doc in `doc.go` say why the usage directory is exempt.
Everything else under `_lym` still counts.

Closes #190
Closes #194

## Design decisions

- The check skips the directory; it does not commit the records first or git-ignore them.
  A commit before the check would still race with a session that ends after it,
  and the exemption is the mechanism the other lym-owned state under `_lym` already uses.
- The usage records stay uncommitted until the next records commit or the run's removal, like the friction notes.

## Tests

`TestCheckResolved_LymOwnStateIsExempt` now also writes an agent session's usage record with `shedrun.WriteSessionUsage`.
It asserts that Fabric's unscoped `Clean` sees the record as a change while the worktree-clean check passes over it,
and that a file under `_lym` outside the exempt state still refuses.

## Merge danger

**Door:** two-way
**Blast radius:** preflight
