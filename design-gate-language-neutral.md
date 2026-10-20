# [.NET 002] Language-neutral gate: lyx gate run takes a slot for any command

`gate-language-neutral` · Note · enhancement, config, webster, shed, dotnet

## Seen (orch, 2026-10-17)

`lyx gate test` runs only `go test`: it takes a hub gate slot, then runs `go test -p <go_parallel>` over the named packages.
A hub in another language has no gate, so its verify command (for example `dotnet test` in a C# hub) runs outside the slot pool.
Several runs then build and test at once with no limit, which is what the slot pool exists to stop.

## Idea

- `lyx gate run -- <cmd>`: take a gate slot, run any command, release the slot.
  The language-specific verbs (`lyx gate test` for Go) become thin wrappers over it.
- Parallelism per language in `language.yaml`, in place of the Go-only `go_parallel`.
- Each language's verify commands and test tiers are declared by the hub, never hard-coded in lyx.

## Operator view, not a decision (2026-10-17)

- lyx is to support Python as well: Go, C# and Python are the three languages in the first round.
- Layered test tiers matter in C# too; see note `dotnet-scoped-verify` for the tiers in the operator's work repository.

- Scoped tests are extremely important: a C# hub cannot run its full suite.

## Orch recommendation, not a decision (2026-10-17)

The slot pool in `internal/gateslot` is already language-neutral (one file lock per slot); only `lyx gate test` on top of it is Go-specific.

1. `lyx gate run -- <cmd>`: take a slot, run any command, release; a nested run inherits the parent slot through the existing environment variable. `lyx gate test` becomes the Go form of it.
2. The hub declares its test tiers in `language.yaml`: name, command and when each runs (card, final, Publish, baseline), with placeholders such as `{parallel}` and `{targets}`, so lyx knows no tool. Examples: `dotnet test --filter {targets} -maxcpucount:{parallel}`, `pytest -n {parallel} {targets}`, `go test -p {parallel} {targets}`. `go_parallel` becomes a general `parallel`.
3. Test selection per language behind one interface: Go has `internal/impactset`; C# needs what note `dotnet-scoped-verify` describes; Python could use `pytest-testmon` or the import graph.
4. The shell guard refuses raw `dotnet test` and `pytest` as it refuses raw module-wide `go test`, so every run goes through the gate.
5. The regression tier (note `regression-baseline-verify`) uses the same tier mechanism.

Order: 1, 2, 4 first; 3 and 5 are the large ones and come separately.

## Related

Notes `dotnet-verify-protocol` and `dotnet-scoped-verify`.
