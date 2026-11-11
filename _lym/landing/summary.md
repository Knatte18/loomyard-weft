# Let await-forks run from below the anchor or in the records sibling

`lym shuttle await-forks` now resolves from any directory under the recorded fabric anchor and from anywhere in the worktree's records sibling.
Before, it took the strict cwd gate of `lymcwd.Resolve`, which accepts only the anchored directory itself.
A session whose harness moved its cwd, for example to the weft's `_lym/plan`, was refused with "cwd is outside the recorded fabric anchor subtree".
It then had to fall back to collecting each fork's result as it arrived.

Closes #110

## Design decisions

- New `lymcwd.ResolveWithin(cwd)` builds the `Location` of the worktree whose anchored directory or records sibling holds `cwd`.
  A cwd in a records sibling resolves to the `Location` of the worktree that sibling belongs to.
  A cwd elsewhere in the worktree stays refused with `ErrCwdOutsideAnchor`, and the error names the anchored directory.
- A name ending in the records-sibling suffix counts as a records sibling only when a worktree of the trimmed name sits beside it.
  Otherwise it resolves as itself.
- `Resolve` and `ResolveWorktree` keep their behaviour.
  The shared anchor lookup is factored into `anchorRelOrRoot`, which `NamedWorktree` and `Resolve` now use as well.
- The `shuttlecli` pre-run picks `ResolveWithin` by command name for `await-forks` only.
  Every other shuttle verb keeps the strict gate.
  The verb is read-only, which is why loosening the gate is safe for it alone.
- The `ErrCwdOutsideAnchor` message changed from "…outside the recorded fabric anchor subtree" to "cwd is not at the recorded fabric anchor".
  The old wording promised a subtree the strict gate never accepted.
  Callers match the sentinel with `errors.Is`, so only the text changes.
- Docs are updated in `docs/overview.md`, `docs/shared-libs/lymcwd.md` and `pattern/PATTERN-cwd-resolution.md`, plus the `await-forks` help text.

## Tests

- `internal/lymcwd/lymcwd_test.go`: `TestResolveWithin` covers the anchored directory, a directory below it, the records sibling, its root,
  and the anchor's records link as accepted, and a sibling directory of the anchor and the worktree root as refused.
  `TestResolveWithin_NoOwnerOrNoMarker` covers a suffixed name with no owner beside it and a checkout with no recorded marker.
- `internal/shuttlecli/resolve_verbs_test.go`: from a records-sibling cwd over a subpath-anchored hub, `await-forks` passes resolution and `state` is refused.

## To check by hand

- The dispatch of `await-forks` to `ResolveWithin` by command name in the `shuttlecli` pre-run (`internal/shuttlecli/cli.go`).
  A review flagged it as untested at class scope; `resolve_verbs_test.go` now covers it from a records-sibling cwd, but nothing covers other strict verbs beyond `state`.
- Run `lym shuttle await-forks` from a session whose cwd is the weft's `_lym/plan`; it was not run end to end in a real burler session.
