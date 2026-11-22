# Hardener live check prefixes bare package paths with ./

The hardener's `harden-live` check failed at setup after every fix round.
The seed pass writes `hardener.packages` as worktree-relative directories without `./`, such as `internal/fabricengine`.
`hardenLiveArgs` in `internal/loomcli/hardenwiring.go` appended them verbatim to `lym gate test`, so go read them as standard-library import paths:
`package internal/fabricengine is not in std`, then `[setup failed]` for each package.
No test ran, and the fixer looped on a gate no fix could clear.

`hardenLiveArgs` now prefixes `./` to each package that `go/build.IsLocalImport` does not recognise as a local path.
A package already written `./internal/x` passes through unchanged.
`CheckCampaign` reads the packages as directories under the worktree root, so both forms stay valid in config and nothing needs migrating.

## Design decisions

- The prefix is applied where the live call is built, not in config, so the seed pass and existing local layers keep working as written.
- `go/build.IsLocalImport` decides what counts as already local, so `.`, `..` and `./x` are left alone.

## To check

- `TestHardenLiveArgs` now expects `./internal/a` for a bare `internal/a` and `./internal/b` for an already-prefixed input.
  The tests were not run in this description step.

## Merge danger

**Door:** two-way
**Blast radius:** hardener
