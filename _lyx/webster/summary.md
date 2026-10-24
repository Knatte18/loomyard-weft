# dotnet-scoped-verify: scoped .NET verify gates

All three batches (13 cards) recorded `done`; the scope guard found no blocking deviation.

## Batch 01-gate-config-keys (head fb8083dd7)

Cards 1-3.
`internal/gateslot` gained the `dotnet` scoper, the `filter`, `shared_build` and `carry` tier keys, the `build` step and `inert_paths`, with their refusals and the inert matcher.
The build summary and timeout lines, the build run record fields and the landing failure record's build flag were added.
`planparser` gained per-target C# seeds for the dotnet scoper's `--card` form.
Deviations (informational): `internal/gateslot/runlog.go`, `internal/planparser/csharpseed.go`.
Departures: none.

## Batch 04-csharp-source-scan (head 8b551f7ef)

Cards 4-9.
New `internal/dotnetimpact` holds the C# lexer and source model, type-name closure, test-class selection, project grouping, filter terms and the diff and card entry points.
`impactset` exports `ChangedPaths` and `AncestorBase`.
`{filter}` renders from the tier filter and invocation terms, and the dotnet scoper is wired into `computeScope` and the per-project invocations.
Deviations (informational): files under `internal/dotnetimpact`, `internal/gatecli/dotnetscope.go`, `scope.go`, `tiers_test.go` and `internal/impactset/targets.go`.
Departures: none.

## Batch 10-fingerprint-and-carry (head 14c48d3cc)

Cards 10-13.
The tier fingerprint covers the new keys and `carry: false` works.
Inert-only diffs run no tier, and whole gates carry over inert-only history.
The shared build runs once per call inside the first opted-in tier's slots.
A real dotnet run test and `docs/dotnet-hub.md` landed, and `pattern/PATTERN-gate-slot.md` was updated.
Deviations (informational): `internal/gatecli/build.go`, `dotnetfixture_integration_test.go`, `green.go`, `green_test.go`, `run.go`, `scope.go`, `test_integration_test.go`, `tierrun.go`, `tiers_test.go`.
Departures: none.

## Verify gate findings

Attempt 1 failed on `github.com/Knatte18/loomyard/cmd/lyx.TestCLIForm_EveryCommand` (test); no recorded card touched that package.


## Verify gate fix commits

The verify-gate fixers' commits above the pre-fix head, and the PATTERN departures their messages declare.

- `290e8332350ec17faaacf89c81eab2fd4d662212` fix: gate test Long back under the 2000-byte cap
