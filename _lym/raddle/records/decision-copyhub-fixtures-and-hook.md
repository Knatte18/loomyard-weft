## Goal

CopyHub in the NewHub fixtures of fabricengine and adoptcli, and the hook without dirname and basename

Takes: newRecordsFixture, the adoptcli hubforge.NewHub sites, post-checkout.sh and hook.go's chainedHookPreamble
Delivers: hubforge.CopyHub of the same anchor at each site whose test is not about the clone or wiring, and shell parameter expansion in the hook and chain wrapper, falling back to . when $0 holds no slash
Exports: nothing

## Scope

The part changes these packages:

- internal/fabricengine (about 25 changed lines)
- internal/adoptcli (about 15 changed lines)

## Decisions

### D5 — `CopyHub` in the `NewHub` fixtures, and the hook without `dirname` and `basename`

- `newRecordsFixture` and the `EndedSet` rows of `TestRecordsReplay`, both in `internal/fabricengine/replay_integration_test.go`, and the `hubforge.NewHub` sites of `internal/adoptcli` take `hubforge.CopyHub` of the same anchor.
  A site whose test is about the clone or the wiring itself keeps `hubforge.NewHub`.
  The `internal/hubreconcile` sites go with D7's `internal/hubreconcile` task, which edits the same file.
- `internal/fabricengine/post-checkout.sh` derives the weft sibling with shell parameter expansion in place of `dirname` and `basename`, and the chain wrapper in `internal/fabricengine/hook.go` derives its directory the same way, falling back to `.` when `$0` holds no slash.
  `git rev-parse --show-toplevel` answers an absolute path with no trailing slash, for which the expansion and the two commands agree.
  A hub that already holds the hook keeps its old script, which still works, since the installer leaves a hook carrying its sentinel alone.

Rationale: a fresh `NewHub` is about 103 ms against 17 ms for a copy; the hook runs on every checkout of a test's pairs, and the two commands are two processes each time.

## Constraints

- `PATTERN-leaf-packages`: the D1 reader keeps `internal/dotgit` standard-library only and spawning nothing; the background's list of what `dotgit` parses gains the worktree entries and refs.
- `PATTERN-gitrepo-client-boundary`: D1 is a local read, which the entry gives to `dotgit`; the porcelain fallback stays on `gitexec`.
- `PATTERN-fabric-boundary`: D1's change to `fabricengine.List` stays inside `internal/fabricengine`; `internal/websterengine` reads its own worktree's siblings through `dotgit`, as it reads them through git today, and through its present porcelain read on a reftable repository.
- `PATTERN-test-isolation`: every template is a hub `fabriccli.CloneAndWire` built plus the build function's steps; no hub is hand-assembled, and no test package wraps `hubforge` in its own fixture type.
  The background's hub-fixture paragraph gains the registered template.
- `PATTERN-test-speed`: each test keeps one `tiergate.Require` at the lowest tier that fits; D8 follows it.

## Auto-mode assumptions

- D1: a reftable repository falls back to the porcelain path.
- D3: the binding fold stays on git, for the reflogs, in place of the entry's "no `amend` and `push --force` per copy".
- D4: the per-repository config layer stays.
- D6: the survey's rows the report did not read again are out.
- D7: `newMergePairFixture` moves only for repeated layers, and the `internal/battencli` `beamHub` joins the moves.

## Open risks

## Acceptance criteria

- D5: the post-checkout hook still warns on a warp and weft branch mismatch and stays silent when they match; the chain wrapper still runs `post-checkout.user` when it is executable.
- D5: Review: each `hubforge.NewHub` site kept names the clone or wiring behavior that needs it.
