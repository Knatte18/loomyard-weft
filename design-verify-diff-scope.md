# Verify gates run only what the diff can break, plus the repo-wide scans

`verify-diff-scope` · Note · enhancement, loom, shed, config

## Seen

On 2026-10-18 the darn run `tokencount-segment-report` changed only `tools/tokencount`, yet its Darn-Write verify gate ran darn.yaml's `verify`, `lyx gate test ./... && lyx gate test --tags integration ./...`: both tiers over the whole module.
The same command runs for every darn and at loom's Merge-In and Publish, whatever the diff touches, so a change to a tool or a test helper waits on the full suite, possibly up to three times.

## Idea

The verify gate runs only what the diff can break: the packages the diff changes, the packages that import them, and every repo-wide check.
The repo-wide checks are the catch: the PATTERN scans and other tests that walk the whole tree must still run for any change, `tools/` included, so the scoped set is "affected packages plus the scan packages", never "affected packages" alone.
lyx must not hard-code Go, so the scoping is a config of the recipe's module, for example a scoped verify command with a placeholder for the package list and a command that derives that list from the diff, with today's whole-module command as the fallback when the derivation fails or the diff touches `go.mod` or shared test infrastructure.
Whether the final gate before landing stays whole-module is a design choice; `dotnet-scoped-verify` asks the same question for a .NET repo.
