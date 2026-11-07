# Registered hub templates in hubforge, built once per test binary

`internal/hubforge` gains `Registration` and `CopyRegistered`.
A test package declares a template once, as a package-level `*Registration` holding a key, a base `Shape` and a build function.
Its template is the base shape's `fabriccli.CloneAndWire` hub with its pairs, plus whatever the build function does on that hub: commits, pushes and pairs under fixed slugs.
None of those start states fits in the closed `Shapes` set.
`CopyRegistered` copies the template through the same relocation, rebase and binding fold `CopyHub` uses.

## Behavior

- **Built once per binary, by key.**
  Each key has its own cache slot with its own lock, and the shared map's lock covers only the lookup.
  So templates of different keys build at once, and parallel first callers of one key share a single build.
- **Refusing TB.**
  The build runs on its own goroutine under a `testing.TB` that records failures.
  `Cleanup`, `Setenv`, `TempDir`, `Chdir`, `ArtifactDir` and every skip fail the build.
  Logging and the remaining methods reach the first caller.
- **Upstream-tip check.**
  After the build function, the board's HEAD must equal the tip of the board's branch in the records bare.
  The tip is read from the bare itself, so a commit pushed into the bare from a clone elsewhere fails the build too.
  The copy's binding fold amends HEAD and force-pushes, and this check keeps that fold from dropping a commit.
- **Failure handling.**
  When a build fails, its first caller fails with the build's errors under the key.
  Every later caller fails naming the key, and the template is never built again.
- **Shared keys.**
  A registration is compared by its address, so when two different registrations use one key, the second caller fails naming the key.

## Design decisions

- The shapes of `Shapes` now build through registrations of their own, keyed by shape name, using the same per-key machinery.
  Before, one mutex covered every shape build.
  As a result, shape builds also run concurrently, record failures, and go through the upstream-tip check.
  `Topology.Add` never commits on the board, so the check holds for the pair-carrying shapes.
  A registration key that equals a shape name, such as `.+pair`, counts as a shared key.
- Besides the five calls the record names, the refusing TB also refuses skips.
  A skip would end the build goroutine without a recorded failure, or mark the first caller skipped from another goroutine.
- The record orders the build steps as build function, then D3's pack, then fingerprint.
  The pack belongs to `hubforge-slim-copy`, which depends on this task, so the fingerprint follows the tip check directly.
- `commitRelocatedBinding` and the package doc now say it amends the board's HEAD commit, not "the one commit".
  A registered template's board can hold more commits than one.

## Docs

- The `internal/hubforge` package doc describes registrations, per-key builds, the refusing TB, the tip check and failure handling.
  It also lists the process-wide calls the TB cannot catch, which review must catch instead.
- The Hub fixtures section of `pattern/PATTERN-test-isolation.md` adds the registered template and the limits on a build function.
  The `PATTERN.md` index line is unchanged.

## Tests

`internal/hubforge/template_test.go`, all at tier 1:

- Two keys built by three parallel callers each.
  The two builds wait on each other, so the test finishes only when they run at once, and each runs exactly once.
- A copy holds the build's board commit and both pairs, and its board is not ahead of its upstream.
  After a commit and push into the copy, the template's fingerprint is unchanged and a later copy lacks the write.
- One table row per failure: a fatal build, each refused TB call, a board commit left unpushed, and a commit pushed into the records bare from a clone inside the template root.
  The first caller's failure names the key and the cause, a later caller's names the key, and each build runs once.
- Two registrations with one key: the second fails naming the key.
