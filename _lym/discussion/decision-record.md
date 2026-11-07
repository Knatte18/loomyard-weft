## Goal

A slimmer hubforge.CopyHub: packed templates, no walk per copy, two-process binding fold

Takes: hubforge-registered-templates' build path in internal/hubforge/template.go
Delivers: CopyHub without the fingerprint walk, SharedHub keeping it, repack and pack-refs at build time for shapes and registrations, and commitRelocatedBinding without its git add
Exports: nothing new

## Scope

The part changes these packages:

- internal/hubforge (about 120 changed lines)

## Decisions

### D3 — A slimmer `CopyHub`

- `hubforge.CopyHub` no longer runs the fingerprint walk; it still fails on a template marked poisoned.
  `hubforge.SharedHub` keeps the walk before and after the test, and poisons the template on a change.
- A template's repositories are packed once at build time with `git repack -ad` and `git pack-refs --all`, after the build function and before the fingerprint, for shapes and registered templates alike.
  A copy then copies a few pack files in place of the loose objects; relocation rewrites only text files, and a pack or `packed-refs` holds no template path.
- The fold of the relocated `.lym-warp` binding stays on git, so HEAD, the branch and the tracking ref keep the reflog entries `internal/fabricengine/reflog.go` reads.
  Its separate `git add` goes: the amend names the binding file, and the push follows, two processes in place of three.

Bound: without the walk on each copy, a stray write by a `hubforge.SharedHub` test can reach copies taken before that test's cleanup poisons the template; copies taken after fail.
Today's `hubforge.SharedHub` sites only read, and this split adds none.
A copy's board keeps the working tree its build left, with HEAD equal to its upstream, its history as built and the binding folded into the tip.

Rationale: copies are about 13 percent of the package's serial time, and the walk and the loose objects are most of each copy.

### Added after Discussion (parent, 2026-11-07): Template repack keeps --cruft

Decision: packRepositories keeps 'git repack -d --cruft' where D3 names 'git repack -ad'; D3's command is read as its aim, a template with no loose object, and nothing further changes for review finding F2 of round 1.

Rationale: 'repack -ad' drops unreachable packed objects and leaves unreachable loose ones loose; --cruft packs them into one cruft pack, so no object a fixture may rely on is lost and none stays loose.

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

- D3: a copy's board HEAD equals its upstream with its history as built, the copy's records bare's branch tip is the board's HEAD, and the tracking ref's newest reflog entry names that SHA.
  The board of a copy of a shape, or of a registration whose build commits what it writes, is also clean.
- D3: a write by a `hubforge.SharedHub` test poisons the template at cleanup, and a later `hubforge.CopyHub` of it fails.
- D3: a template's repositories hold no loose objects after the build, and a `hubforge.SharedHub` taken right after the build sees no change.
