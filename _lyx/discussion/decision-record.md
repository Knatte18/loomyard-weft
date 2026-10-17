# Decision record — stencilstore dev-build warning on creel verbs

## Goal

No `lyx creel` verb prints the `stencilstore: dev build does not refresh untouched stencils` warning, or any other stencil-seed log line, on its stderr.
The stderr of a creel verb carries only creel's own refusals and stall progress lines.

## Scope

- In: the `lyx creel` group and every command under it, the hidden `daemon` included, decline the root pre-run's stencil-seed pass.
- In: the seed-gate test pins that every creel command carries the opt-out.
- In: the `internal/creelcli` package doc states that creel skips the seed pass.
- Out: the wording, level and once-per-pass behavior of the stencilstore refresh notices themselves.
- Out: the seed pass of every other command group.

## Decisions

### Creel commands opt out of the seed pass

Decision: `creelcli.Command()` sets `clihelp.SkipStencilSeedAnnotation` to `clihelp.AnnotationEnabled` on the `creel` group and on each of its subcommands.
Rationale: the warning prints only from a seed pass that writes, and the root pre-run runs that pass before every command, so the first creel verb in a hub whose board stencils are due prints it.
Creel reads no stencils, so the pass is waste for it, and its daemon is a long-lived process that should never commit seed writes to the hub, the reason reed's `switch` and `follow` already opt out.
Bound: the opt-out skips only the seed pass for creel commands; the next stencil-reading command in the hub seeds as before, and no other command's pass changes.

### The annotation goes on each command, not through a parent walk

Decision: the opt-out is set on every creel command in a loop over the group and its children, and `skipStencilSeed` keeps reading only the running command's own annotations.
Rationale: the annotation's contract is per command, and a parent walk would change it for every group in the binary.

## Constraints

- `PATTERN-cli-cobra`: creel stays one cobra subtree under the root.
- `PATTERN-test-economy`: the coverage extends `TestSkipStencilSeed_HonoursTheAnnotation` with rows, never a new top-level test.
- `PATTERN-test-speed`: the test only builds cobra values and runs no hook.

## Auto-mode assumptions

- The reported "every verb" is the first creel verb of a freshly built sandbox hub, whose seed pass writes the stencils the board lacks; the sandbox traces hold the warning once per re-test, on that first verb.
  Opting creel out covers both readings of the report.

## Open risks

- A hub whose only commands are creel verbs never seeds its board stencils until another command runs; no creel path reads them, so nothing depends on that.

## Acceptance criteria

- Every command in `creelcli.Command()`'s tree, the group itself and the hidden `daemon` included, makes `skipStencilSeed` report true.
- The existing rows of the seed-gate test still pass: an absent, unrelated or `"false"` annotation does not opt out.
- `lyx gate test ./cmd/lyx ./internal/creelcli` passes.

## Notes for the plan writer

- `skipStencilSeed` in `cmd/lyx/stencilseed.go` is the predicate the root pre-run consults; `internal/reedcli/switch.go` shows the existing opt-out.
