# hubforge: registered hub templates built once per test binary

`internal/hubforge` gains `Registration`, a package-level value a test package declares to get a hub template the closed `Shapes` set cannot hold.
It carries a `Key`, a base `Shape` and a `Build` function that brings the hub to a start state, such as commits, pushes and pairs under fixed slugs.
`CopyRegistered` copies it into a test's own temp directory, through the same relocation and rebase as `CopyHub`.
A registered template is built once per test binary on first use, so a test pays the build once and a cheap copy after that.
`pattern/PATTERN-test-isolation.md` gains the registered template in its hub-fixture background.

## Design

- The template is built the way a shape's template is: its own `lymtemp` root, the bares, `fabriccli.CloneAndWire`, the base shape's pairs, then `Build` on that hub; never from a copy.
  The fingerprint is taken after the build.
- Each shape of `Shapes` is now built through a registration of its own, keyed by the shape's name, so `CopyHub` and `SharedHub` share the one build path.
- Builds lock per key, so templates of different keys build at once.
  The shared map takes its own short lock.
- Two different registrations with one key fail the second caller naming the key; registrations are compared by address.
- A failed build fails its first caller with the build's error.
  Every later caller fails naming the key, and the template is not rebuilt.
- `Build` runs on its own goroutine under a `testing.TB` that records failures.
  `Cleanup`, `Setenv`, `TempDir`, `Chdir`, `ArtifactDir` and every skip fail the build naming the key, so no template state is tied to the first caller or reaches a path outside the template root.
- After the build, the board's HEAD must equal the tip of its branch in the records bare, read from the bare and not from the board's tracking ref.
  A copy folds its relocated binding into the board's HEAD and force-pushes it, which would drop any commit the bare holds beyond that HEAD.
  `commitRelocatedBinding` now keeps the board's whole history instead of a single commit.

## Limits

The refusing TB cannot see a build's process-wide calls: `os.Setenv`, `os.MkdirTemp`, `git config --global`, a package-global seam setter, a once-only registration such as `boardkit.InstallReplay`, or code a Bolt replay changes.
These leak into every later copy or test in the binary, and `os.MkdirTemp` leaves a path that relocation does not rewrite.
Enforcement is review discipline, stated in `Registration.Build`, the package doc and the pattern.
A test that installs a process-global seam takes its copies before installing it.

## To check by hand

- Every future registration's `Build` makes none of the process-wide calls listed under Limits.
- Several templates building together at a binary's start raise the load there; the gain assumes those builds no longer queue.
