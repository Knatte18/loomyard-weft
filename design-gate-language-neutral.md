# Language-neutral gate: lyx gate run takes a slot for any command

`gate-language-neutral` · Note · enhancement, config, webster, shed

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

## Related

Notes `dotnet-verify-protocol` and `dotnet-scoped-verify`.
