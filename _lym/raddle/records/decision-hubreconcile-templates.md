## Goal

internal/hubreconcile fixtures onto registered templates and CopyHub, with the held-lock test at tier 0

Takes: hubforge-registered-templates' registration
Delivers: newStaleHub with pair-a built by a registration, the package's hubforge.NewHub sites on hubforge.CopyHub per D5's rule, and TestEnsure_HeldLockTimesOutWithoutStamping rewritten onto in-memory inputs at tier 0
Exports: nothing

## Scope

The part changes these packages:

- internal/hubreconcile (about 120 changed lines)

## Decisions

### D5 — `CopyHub` in the `NewHub` fixtures, and the hook without `dirname` and `basename`

- `newRecordsFixture` and the `EndedSet` rows of `TestRecordsReplay`, both in `internal/fabricengine/replay_integration_test.go`, and the `hubforge.NewHub` sites of `internal/adoptcli` take `hubforge.CopyHub` of the same anchor.
  A site whose test is about the clone or the wiring itself keeps `hubforge.NewHub`.
  The `internal/hubreconcile` sites go with D7's `internal/hubreconcile` task, which edits the same file.
- `internal/fabricengine/post-checkout.sh` derives the weft sibling with shell parameter expansion in place of `dirname` and `basename`, and the chain wrapper in `internal/fabricengine/hook.go` derives its directory the same way, falling back to `.` when `$0` holds no slash.
  `git rev-parse --show-toplevel` answers an absolute path with no trailing slash, for which the expansion and the two commands agree.
  A hub that already holds the hook keeps its old script, which still works, since the installer leaves a hook carrying its sentinel alone.

Rationale: a fresh `NewHub` is about 103 ms against 17 ms for a copy; the hook runs on every checkout of a test's pairs, and the two commands are two processes each time.

### D7 — Fixtures onto registered templates, one package per task

A start state moves onto a D2 template when at least three sites build it; the builder is the fixture's present build steps, moved as they are, so the start state is the same by construction.
Tests that exercise `Add` keep adding their pairs, and a pair whose slug a test chooses per row stays added in the test.
A step that cannot run in a build stays in the test after the copy:
one that places a path outside the hub, one that makes a process-wide call D2's Bound names, one that leaves the board's HEAD unequal to its upstream tip, ahead or behind,
or one that pushes into a bare from anywhere but the board, as `endedHub`'s `originBoard` pushes into the records bare through a clone outside the hub.

- `internal/fabricengine`, anchor and origin: `newAnchorHub` and `newOriginHub`, with `originPair` and `endedHub` where a fixed slug set repeats at least three times; `endedHub`'s `originBoard` runs on the copy.
- `internal/fabricengine`, live state and merge pairs:
  - Each verb's arrange in the cross product splits into a build, registered per verb and anchor, and a derivation of the verb's fixture value from the copy's `Hub`.
  - An arrange that places a directory outside the hub stays whole in the test.
  - The `newMergePairFixture` layers whose branch commits repeat at least three times move; `newMergePairFixture` itself is already a plain copy and gains nothing alone.
- `internal/hubreconcile`: `newStaleHub` with its fixed slug `pair-a`, and the package's `hubforge.NewHub` sites onto `hubforge.CopyHub` per D5's rule.
- `internal/loomcli`: `beamHub` and `newBadReedUpFixture`; the latter's base, a hub with the corrupted reed line and the pair, is the template, and each site's seed runs on its copy.
- `internal/battencli`: its own `beamHub`, a beam, a member and a plain pair.

Rationale: 9 to 11 s of a pass (report, section 2, rank 1), with 2 to 4 s more across the other packages.

### D8 — Tier 0 moves as each file is touched

A task of this split that edits a file holding one of the report's section 4 tier 1 tests that run no git moves that test to tier 0, as `PATTERN-test-speed` places it.
In this split that is `TestEnsure_HeldLockTimesOutWithoutStamping` in the `internal/hubreconcile` task, rewritten onto in-memory inputs with its assertions kept.
No task opens a file only to move a test, so `TestNoticeSite` stays at tier 1: the `internal/battencli` task does not edit `internal/battencli/notices_integration_test.go`.

Rationale: under 1 s of wall in all, so the move rides on edits that happen anyway.

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

- D5: Review: each `hubforge.NewHub` site kept names the clone or wiring behavior that needs it.
- D7: each moved package's tests pass with the same assertions, and the tests that exercise `Add` still add their pairs.
- D7: Review: no registration's build calls `os.Setenv`, `os.MkdirTemp` or `git config --global`, keeps the TB's `Context()` past the build, calls a package-global seam setter or once-only registration, or reaches code whose result a once-only registration changes.
- D7: Review: each test that installs a process-global seam takes its copies before installing it.
- D8: each moved test runs at tier 0 with its assertions kept.
