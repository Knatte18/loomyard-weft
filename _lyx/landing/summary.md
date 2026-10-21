# Deploy pushes no prod pointer and skips a foreign lyx on PATH

`update-plugins.sh` (`tools/deploy`) stops pushing the shared `prod` pointer, and its other-lyx-on-PATH warning names only copies of lyx itself, each file once (issue #766).

## What changes

- **No `prod` push.**
  A production deploy no longer pushes `HEAD` to `origin/prod`, and the ancestry check that kept `prod` moving forward goes with it.
  Each machine deploys on its own, so one pointer on origin could not say what any machine runs.
  The refusals of a dirty tree and of a HEAD not on `origin/main` stay; the latter now lives in `pushedGate`.
- **Foreign `lyx` on PATH.**
  `otherLyxBinaries` (formerly `otherBinaries`) reports a file named `lyx` outside the Go bin dir only when `debug/buildinfo` reads it and its main package is `github.com/Knatte18/loomyard/cmd/lyx`.
  The Ubuntu LyX document processor at `/usr/bin/lyx`, or any non-Go program of that name, is no longer flagged for removal.
- **One file, one warning.**
  Each candidate resolves through `filepath.EvalSymlinks`, and a file already seen is skipped, so `/bin` linking to `/usr/bin` reports once, and a PATH alias of the Go bin dir never reports the production binary itself.
- `CLAUDE.md`'s Production section and `README.md` stop saying the script moves `prod`.

## Design decisions

- Build info rather than a version probe identifies lyx: reading it runs nothing, where `lyx --version` would start an unknown program from PATH.
  A lyx built from a fork with a different module path is no longer reported.
- The existing `origin/prod` branch is left in place; deleting it is a one-time operator action.
  The hub-wide orch rules on Bolt are outside this diff.
- `pushedGate` has no unit test: its one decision is the result of `git merge-base --is-ancestor`, and the package's tests spawn no git.

## Tests

`TestOtherLyxBinaries_ReportsEachLyxCopyOutsideTheDestDirOnce` replaces the fake-filesystem test with a real temp layout: copies of the test binary stand in for lyx, matched by its own build-info path, beside a shell script named `lyx`, a missing directory and `fslink` directory links aliasing a copy and the Go bin dir.
A second row with another main package path expects no report.
The `prod` ancestry test is removed with the check it covered.
