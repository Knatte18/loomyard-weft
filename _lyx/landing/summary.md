# Creel commands skip the stencil-seed pass

A dev-build `lyx` printed `stencilstore: dev build does not refresh untouched stencils ...` on stderr from the first `lyx creel` verb of a hub whose board stencils were due.
The root pre-run seeds stencils before every command, and the warning comes from a seed pass that writes.
The line mixed into the stderr that creel verbs use for refusals and stall progress.

Every `lyx creel` command now opts out of that pass, so a creel verb's stderr carries only creel's own lines.

Closes #702

## Design decisions

- `creelcli.Command()` sets `clihelp.SkipStencilSeedAnnotation` to `clihelp.AnnotationEnabled` on the `creel` group and on each of its subcommands, the hidden `daemon` included.
  Creel reads no stencils, and its long-lived daemon should never commit seed writes into the hub, the same reason `reed switch` and `reed follow` already opt out.
- The annotation is set on each command in a loop, not through a parent walk.
  `skipStencilSeed` still reads only the running command's own annotations, so no other command group's seed pass changes.
- The wording, level and once-per-pass behavior of the stencilstore notices are untouched.
- A hub whose only commands are creel verbs never seeds its board stencils until another command runs.
  No creel path reads them.

## Tests and docs

- `TestSkipStencilSeed_HonoursTheAnnotation` in `cmd/lyx/stencilseedgate_test.go` gains one row per creel command, the group included, each expecting the opt-out.
  The existing absent, unrelated and `"false"` annotation rows are unchanged.
- The `internal/creelcli` package doc states that creel skips the seed pass.

## To check by hand

Not run: a dev build (`./deploy-dev`) in a fresh hub, confirming that the first `lyx creel` verb prints no `stencilstore:` line on stderr.
