## Goal

Registered hub templates in hubforge, built once per test binary

Takes: hubforge's shape templates, relocation and copy in internal/hubforge/template.go
Delivers: a registration of key, base Shape and build function, built once per binary under a lock per key with a refusing testing.TB and the upstream-tip check, and a copy call that takes it; the PATTERN-test-isolation background gains the registered template
Exports: the registration type and its copy call in internal/hubforge

## Scope

The part changes these packages:

- internal/hubforge (about 450 changed lines)

## Decisions

### D2 — Registered hub templates in `hubforge`

`internal/hubforge` gains a registration, a package-level value holding a key, a base `Shape` and a build function, and a copy call that takes the registration.

- The template is built the way a shape's template is built today, in its own `lymtemp` root: the bares, the clone and the base shape's pairs, then the build function on that hub; never from a copy.
  Then come D3's pack and the fingerprint, in that order.
- It is built once per test binary on first use, and copied through the same relocation and rebase as a shape's copy, through a lookup by key beside the closed `Shapes` set.
- The build function returns nothing and captures no per-call data: a test derives the paths it needs from the copy's `Hub`, and a start state that varies needs its own registration.
- Builds take a lock per key, so templates of different keys build at once; the shared map takes its own short lock, and the existing ref-counted fixture-build mark already allows concurrent builds.
- Two different registrations with one key fail the second caller naming the key; a registration is compared by identity.
- A build that fails marks the template failed: its first caller fails with the build's error, and every later caller fails naming the key without rebuilding.
- The build function gets a `testing.TB` whose `Cleanup`, `Setenv`, `TempDir`, `Chdir` and `ArtifactDir` fail the build naming the key.
  So no build ties template state to its first caller's lifetime, or reaches a path outside the template root through the TB.
  The helpers a shape build uses today, `AddPair`, `AddPairWith` and the `gitkit` commit helpers, call none of them.
- After a build, the board's HEAD equals the upstream tip: the tip of the board's current branch in the records bare, read from the bare, never the board's tracking ref.
  When the board's HEAD differs from it, the build fails naming the key,
  so D3's fold and push never drop a commit the builder made, on the board or pushed into the bare from elsewhere.

Bound: a template built wrongly fails every test that copies it, loudly and naming the key; a test that wrote to its copy touches nothing shared, as with a shape's copy.
The refusing TB does not catch a build's process-wide calls made outside it:
`os.Setenv`, `os.MkdirTemp`, `git config --global` into the binary's one hermetic global config, or the TB's `Context()` kept past the build.
Relocation rewrites only the template root, so a path `os.MkdirTemp` leaves in the template reaches every copy unflagged.
`os.Setenv` and `git config --global` leave no path in the template but change state every later test in the binary reads.
A build also runs under whatever process-global seams and registrations are installed when its first caller reaches it,
and a once-only registration it makes, such as `boardkit.InstallReplay`, is never undone; either result reaches every later copy or test.
So a build calls no package-global seam setter or once-only registration,
and reaches no code whose result a once-only registration changes, such as a Bolt replay;
a test that installs a process-global seam takes its copies before installing it.
Review bounds them: the review of each registration in its D7 task checks that its build makes none of those calls, and that its tests keep that order.

Rationale: the fixtures' start states are a hub plus commits, pushes and pairs under fixed slugs, which the closed shape set cannot hold; a registered builder holds any of them, built once at about 0.18 s and copied at about 17 ms, and pays from its third use.

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

- D2's builds by key at once raise the load at a binary's start, where several templates build together; the gain assumes those builds no longer queue.

## Acceptance criteria

- D2: two registered templates of different keys build concurrently, each exactly once per binary, under parallel first callers.
- D2: a copy of a registered template holds the build function's commits and pairs, and a write to the copy leaves the template unchanged.
- D2: a build that fails fails every later caller naming the key; a build function that calls `Cleanup`, `Setenv`, `TempDir`, `Chdir` or `ArtifactDir`, or leaves the board's HEAD unequal to its branch's tip in the records bare, fails naming the key.
- D2: a build that, after the board's push, pushes a commit into the records bare from a clone inside the template root fails naming the key.
- D2: two different registrations with one key fail naming the key.

## PATTERN changes

- `PATTERN-test-isolation` — changes: Testing package: hermetic git and `tmuxkit.Main`; shared support in one `internal/testkit/<kit>/`; hub fixtures via `hubforge`. (test)
