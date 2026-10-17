# Creel commands skip the stencil-seed pass

Fixes #702.

The root pre-run runs the stencil-seed pass before every command.
When the pass writes, `stencilstore.Reconcile` logs its refresh notices, and under a dev build that is the Warn `stencilstore: dev build does not refresh untouched stencils ...` on stderr.
The first creel verb in a hub whose board stencils were due printed that line into the stderr creel uses for refusals and stall progress lines.
In the creel re-tests this was the first verb of the freshly built sandbox hub.

Creel reads no stencils, so the pass is waste for it.
Its daemon is a long-lived detached process that should never commit seed writes into the hub, the reason reed's `switch` and `follow` already opt out.

## Changes

- `internal/creelcli/cli.go`: `Command()` sets `clihelp.SkipStencilSeedAnnotation` on the `creel` group and on each of its subcommands, the hidden `daemon` included.
- `internal/creelcli/doc.go`: the package doc states that every creel command skips the seed pass.
- `cmd/lyx/stencilseedgate_test.go`: `TestSkipStencilSeed_HonoursTheAnnotation` gains one row per creel command, built from `creelcli.Command()`, so a new creel verb is covered without editing the test.

## Design decisions

- The annotation is set on every creel command in a loop, and `skipStencilSeed` still reads only the running command's own annotations.
  A parent walk would have changed the per-command contract for every group in the binary.
- The stencilstore notices keep their level and wording: for a stencil-reading verb, the dev-build warning still tells a tester that producers read the older on-disk copies.
- A hub where only creel verbs run seeds its board stencils at the next stencil-reading command, as before.

## Testing

- `lyx gate test ./cmd/lyx ./internal/creelcli`
- `lyx gate test --tags integration ./internal/creelcli ./cmd/lyx`
