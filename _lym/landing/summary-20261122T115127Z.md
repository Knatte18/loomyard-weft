# Hardener live check passes packages go can resolve

`hardenLiveArgs` in `internal/loomcli/hardenwiring.go` builds the `lym gate test --gate harden-live` call a hardener run makes after each fix round.
It appended `hardener.packages` entries verbatim.
An entry written as a bare worktree-relative directory, such as `internal/fabricengine`, reached go as an import path:
go looked it up in the standard library, failed with `package internal/fabricengine is not in std`, and every tier reported `[setup failed]` before any test ran.
The live check was red every round, and the fixer looped on a failure no fix could clear.

## Change

`hardenLiveArgs` now passes each package as a path go reads as a directory.
An entry `go/build.IsLocalImport` rejects gets a `./` prefix;
an entry already starting with `./` or `../`, or equal to `.` or `..`, passes unchanged.
`hardenshed.CheckCampaign` already reads the entries as directories under the worktree root, so both forms stay valid in `hardener.yaml`.

## Tests

`TestHardenLiveArgs` now feeds one bare entry (`internal/a`) and one prefixed entry (`./internal/b`) and expects `./internal/a` and `./internal/b` in both the plain and the `--copies` call.
On the merge base the bare entry comes back unprefixed, so the test fails there.

## Merge danger

**Door:** two-way
**Blast radius:** hardener
