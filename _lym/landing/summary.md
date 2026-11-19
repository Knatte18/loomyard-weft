# stencilstore: prefer shipped stencils over a newer binary's board copy

A run halted `failed` at Darn-Write when its gate fixer's prompt build reported `include "loom-fragment-loosened-check" has no entry in the includes` (issue #149).
The include wiring itself was fixed by 72c3e192: `stencilstore.ReadForEngine` writes plain includes in place, and `TestFixerSpec` (`internal/loomcli`) and `TestRunWithFixer` (`internal/gatecli`) already render every gatefix body from the shipped stencils.
The failure came from version skew: a driver built before 72c3e192 read a `gatefix-body-darn` board copy that a newer production binary had refreshed under it, and its own code could not resolve the new include.
This change stops that class of failure for any long-running process.

## What changes

- `contracts/stencils` registers its embedded defaults with `stencilstore.RegisterShipped` at package init.
- `stencilstore.Read`, and the engine-block read behind `ReadSet` and `ReadForEngine`, return the running binary's shipped default in place of a board copy when all of these hold:
  the copy is untouched (its stamp matches its body), its recorded writer has a later commit time than the running binary, and its body differs from the shipped default.
- The first such read of each stencil in a process logs a warning naming the stencil, the writer and running revisions, and the way forward: restart the run's driver on the current lym binary.
- The package doc of `internal/stencilstore` describes the fallback.

## Design decisions

- Fallback instead of refusal: an untouched copy holds no operator edit, so rendering the binary's own text loses nothing and keeps the run alive; a refusal would halt every row that reads a refreshed stencil.
- An edited board copy is read as it stands, even when a newer binary's writer is recorded, so an operator's text is never replaced.
- Ordering is by commit time because the read path runs no git; a writer or running binary with no recorded time is never treated as newer.
  That keeps test binaries and copies seeded without a writer on the existing path.
- The registry is registered at init rather than passed to `Read`, since every renderer calls `Read` with only a directory and a name.

## To check by hand

- `TestShippedOverNewer` pins the decision function; the wiring through `Read` reads `buildvcs.Running()` and is not exercised by a test, since test binaries carry no VCS stamp.
