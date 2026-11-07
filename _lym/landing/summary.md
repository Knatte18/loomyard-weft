# fabricengine tests: anchor and origin fixtures onto registered templates

The `internal/fabricengine` integration tests now build their anchor and origin hubs once per package and copy them per test, instead of rebuilding them at every site.
This cuts the per-pass cost of the fixtures that three or more sites repeated.
Only test files change; no production code and no export changes.

- `newAnchorHub` copies `anchorTemplate`, the "." hub whose prime warp tracks `sub/` and `other/`.
- `newOriginHub` copies `originTemplate`, the "." hub with the prime's records branch published.
- `newOriginTaskHub` copies `originTaskTemplate`, which adds the pair `task` with its upstream set.
  It replaces the `newOriginHub` plus `originPair(…, "task")` sequence that repeated across the Judge and Ended tests.
- `newMovesPair` copies `movesTemplate`: the `task` pair, a `Land` run record at the prime, and the pair's records branch published.
- `endedHub` copies `endedTemplate`: the `task` pair with a run record and a published board that holds `task` live.
  The origin board's done mark (`originBoard`) runs on the copy, since it pushes into the records bare through a clone outside the hub.
- Each template's build is the fixture's former build steps moved unchanged, so the start state is the same by construction.
- Helpers a build calls (`originPair`, `writeStatus`, `publishBranch`, `publishBoard`, `boardFileOf`) take `testing.TB` instead of `*testing.T`.

## Design decisions

- `endedHub` no longer takes slugs: its fixed slug is `task`.
  The one test that needs `task` and `two` builds from `newOriginHub` and calls `arrangeEnded` directly, because its slug set is not repeated.
- Tests that exercise `Add` still add their pairs, and a pair whose slug a test picks per row is still added in the test.
- No build calls `os.Setenv`, `os.MkdirTemp` or `git config --global`, keeps the TB's `Context()` past the build, or calls a package-global seam setter or once-only registration.
- `TestWorktreeSeam_Events` installs a process-global worktree seam.
  It now takes every template copy it needs before installing the seam, so a template's first build never runs under the fake seam.
  The "." shape is copied once first, because the reconcile fixtures copy it inside `buildReconcileFixture`.
  `boardkit.InstallReplay` moved up with those copies.

## To check by hand

- `seam-installed-before-template-copies` (design, blocking in review): `TestWorktreeSeam_Events` in `internal/fabricengine/worktreeseam_integration_test.go`.
  Confirm every later subtest's template copy, direct or inside a fixture builder, is taken before `InstallWorktreeSeam`, and that the pre-copied hubs are not mutated by an earlier subtest.
- Run the package's integration tests, such as `lym gate test ./internal/fabricengine`, and compare the assertions with the previous ones; none was run for this description.
