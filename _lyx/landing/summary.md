# Deploy: drop the prod pointer, skip foreign lyx on PATH

`update-plugins.sh` (`tools/deploy`) no longer pushes `HEAD` to `origin/prod`, and its other-lyx-on-PATH warning names only copies of lyx itself, each file once.

`prod` was one pointer on origin shared by every machine, while each machine's installed binary is local.
After a deploy on one machine it named a commit the other machine did not run, so it said nothing about what either ran.
Only `tools/deploy` wrote it, and no lyx code reads it.

The PATH check printed `WARNING: another lyx on PATH: /usr/bin/lyx - remove it` for the Ubuntu LyX document processor, and again for `/bin/lyx`, which is the same file through a symlinked `/bin`.

Closes #766

## What changes

- A production deploy pushes nothing to origin and prints no `Moved prod` line.
- A deploy still refuses a dirty tree and a `HEAD` that is not on `origin/main` after a fetch.
  Only the `prod` ancestry half of the gate is gone.
- The PATH scan reports a `lyx` outside the Go bin dir only when it is a copy of lyx, before or after the production binary on PATH.
- A file reached through several PATH directories is reported once, and a PATH alias of the Go bin dir reports nothing.
- `CLAUDE.md`'s Production section and `README.md`'s production paragraph stop saying the script moves `prod`.

## Design decisions

- **Recognition by Go build info.**
  A PATH binary counts as lyx when `debug/buildinfo` reads it and its main package path equals `lyxMainPackage` (`github.com/Knatte18/loomyard/cmd/lyx`).
  A non-Go binary, or a Go binary of another main package, is foreign and not reported.
  Build info is embedded in every Go binary and reading it runs nothing, unlike a version probe that would execute an unknown program.
  A lyx built from a fork with a different module path is no longer reported.
- **Deduplication by resolved file.**
  Each candidate goes through `filepath.EvalSymlinks`, and a file whose resolved path was already seen is skipped, the production binary's own path included.
  A Windows alias through a junction is deduplicated only as far as `filepath.EvalSymlinks` follows junctions.
- **No new warning for a foreign `lyx` that shadows production.**
  The board entry asks only that foreign binaries not be flagged for removal.

## To do by hand

- The existing `origin/prod` branch is not deleted; delete it on origin when wanted.
- The hub-wide orch rules on Bolt are outside this diff; update them if they name `prod`.
- No production deploy was run; `update-plugins.sh` runs only when the operator runs it.
