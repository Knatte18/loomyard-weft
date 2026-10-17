# Creel commands skip the root pre-run's stencil-seed pass

No `lyx creel` verb prints the `stencilstore: dev build does not refresh untouched stencils ...` warning any more.
The warning came from the root pre-run's stencil-seed pass, which runs before every command and prints it when it writes board stencils that are due.
The first creel verb in a hub with due stencils therefore printed it on stderr, mixed into creel's refusals and stall progress lines.
Creel reads no stencils, so the pass is wasted on it.
The creel daemon is long-lived and should not commit seed writes into the hub, the reason `reed switch` and `reed follow` already opt out.

`creelcli.Command()` now sets `clihelp.SkipStencilSeedAnnotation` to `clihelp.AnnotationEnabled` on the `creel` group and on each of its subcommands, the hidden `daemon` included.
`internal/creelcli/doc.go` states the skip.
`TestSkipStencilSeed_HonoursTheAnnotation` gains one row per creel command, so a verb added without the opt-out fails the test.

Closes #702

## Design decisions

- The annotation is set on each command in a loop, not inherited through a parent walk.
  `skipStencilSeed` keeps reading only the running command's own annotations; a parent walk would change that contract for every group in the binary.
- Only creel's pass is skipped.
  The next stencil-reading command in the hub seeds as before, and no other command group changes.
- The wording, level and once-per-pass behavior of the stencilstore notices are unchanged.
- A hub whose only commands are creel verbs never seeds its board stencils until another command runs.
  No creel path reads them, so nothing depends on it.

## To check by hand

- The report was seen with a dev binary from `./deploy-dev`.
  Running a creel verb in a freshly built sandbox hub with due board stencils, and confirming stderr is clean, was not done; the tests build cobra values and run no hook.
