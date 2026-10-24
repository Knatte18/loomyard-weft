# Fixer-report, round 1

## Changes

All fixed as proposed. F1–F4 went into one commit, not one per finding, since the edits share `cluster.go` and its test table.

- F1: `resolveShellPath` (`internal/burlerengine/cluster.go`) no longer clamps a `..` that climbs above a `~` or `$` anchor; it returns `../` followed by the path, which lies inside no recorded root. Row added: `git init "$S"` then `git -C "$S/../target" commit -m x` is returned.
- F2: `record` refuses a relative created path that is `.`, `..` or starts with `../`. Row added: `git init ../x` then `git -C ../x commit -m x` is returned.
- F3: rows added for the `--` terminator, the valued options of `git init` and `git worktree add`, a trailing slash, and an anchored `cd` after a bare `cd`.
- F4: `internal/burlerengine/doc.go` states the `git init` bound.

- Gate repair: the `record` comment in `cluster.go` was wrapped at a fixed column; it is now broken at sentence boundaries, one sentence per line.
  The `internal/burlerengine` tests pass.

## Self-check

- fix-consistency, LOW: the bound wording in `cluster.go` and `doc.go` understates the `record` rule. Fixed as proposed (both texts now name the `..` bounds).
- fix-proposal, LOW: same finding as above. Merged with it and fixed with it.
- fix-code-correctness, LOW: the `--` row passes without the terminator handling. Fixed: the row is now `git init -- -r` then `git -C -r commit -m x`.
- fix-code-correctness, LOW: same wording finding as the first. Merged with it.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
