# Card 21 — round-gate-tier3

**Create:**
- `plan:internal/loomshed#checkedPublishFailure` -> `func checkedPublishFailure(paths verifytree.Paths, worktreeRoot string) (verifytree.PublishFailure, bool)`
- `plan:internal/loomshed#publishFailureCommand` -> `func publishFailureCommand(f verifytree.PublishFailure, packages []string) string`
- `internal/loomshed/publishfailure_test.go`
- `internal/loomshed/publishfailure_integration_test.go`

**Edit:**
- `internal/loomshed#NewVerifyGate`
- `internal/impactset#Derivation`
- `internal/impactset#Derive`
- `internal/impactset#commandFor` -> `func commandFor(graph moduleGraph, changedPaths []string, guards map[string][]string) (command string, packages []string, fallback string)`
- `internal/impactset/derive_test.go`
- `internal/loomshed/verifygate_integration_test.go`
- `internal/impactset/doc.go`
- `internal/loomshed/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/verifytree#ReadPublishFailure`
- `internal/gitrepo#Repo.CommitParents`
- `internal/gitrepo#New`

**Intent:**
Run tier 3 at the Webster-Burler round gate only to confirm a fix of a Publish failure (D11).
`checkedPublishFailure` reads the record and keeps only the fields that pass D12's shape checks, logging each field it drops: the merge-in commit must be a full hex object name that resolves through `gitrepo`'s go-git reads (`CommitParents`) to a commit with two or more parents in the worktree; each test path must be a top-level `Test` identifier followed by optional `/`-separated subtest segments holding no whitespace, quote or shell metacharacter; the log path must lie in the verify directory, the directory Publish writes its verify logs to.
The kind renders from the record's closed set, never as text, and an unknown kind drops the record.
`publishFailureCommand` builds the extra steps from those checked fields alone, never running text from the record as a shell command: each failing test run by name in its package under the failing verify's tier (`go test -tags integration` for the plan-verify kind, `go test -tags tmux` for `publish_verify`), with `-run` anchoring each segment through `regexp.QuoteMeta` and single-quoted the way `impactset` quotes its guard runs; and, for the `publish_verify` kind, `go test -tags tmux` over `packages`, the impacted set.
`Derivation` gains `Packages`, the impacted set's package arguments, empty with a fallback; `commandFor` returns them beside the command and `Derive` sets them.
`NewVerifyGate` appends the extra steps with ` && ` to its round command when a checked record is present, using `./...` as the tmux pass's packages when the derivation fell back to the plan verify; with no record the round gate runs no tier 3, as today.
An empty impacted set with no fallback (only non-Go files changed) leaves `Packages` empty: `publishFailureCommand` then skips the impacted-set `tmux` pass and keeps the named-test steps, never rendering a package-less `go test -tags tmux`.
A record with no surviving test (a build failure, a panic before any test line) makes the gate run the impacted-set pass alone.
Publish reruns both verifies in full regardless, and stays the guard.
Bound: the record sits in the pair's own `.lyx/`, so deleting or editing it can skip, narrow or redirect this pass; after the shape checks, an edited record carries no shell text into the gate: what still passes is a well-formed test name that never failed, another real merge commit of the worktree, and a log path naming another file in the verify directory, whose content a pair session can rewrite, as it can the log copy itself; the gate never reads the log, so only the review prompt (next card) carries that path, and Publish's full rerun bounds all three (D11's Bound).
`publishfailure_test.go` (tier 1) is table-driven over what needs no repository: a merge field that is `HEAD --output=x` or a short hash, a test name holding a metacharacter and a log path outside the verify directory each drop that field, and the command for each kind with and without tests and, for `publish_verify`, with an empty package list.
`publishfailure_integration_test.go` (integration tier, since it builds a scratch repository) holds the git-reading rows in one table-driven test: a full hex name of a real merge commit is kept, and a full hex name of a non-merge commit is dropped.
The `internal/loomshed` budget row rises by two for these two tests.
`verifygate_integration_test.go` gains rows: a `publish_verify` record makes the gate run the named tests and the impacted-set `tmux` pass; no record runs no `tmux` test.
`derive_test.go` asserts the returned packages; `impactset`'s doc names the field.
`internal/loomshed/doc.go`'s round-gate paragraph, which says the round compiles the `tmux` and `llm` tiers and never runs them, is rewritten: the round runs the `tmux` tier only while a checked Publish failure record is present, as the failing tests and the impacted-set pass above, and never runs `llm`.

**ImpactSummary:** The round gate's command grows only while a Publish failure record is present; impactset's derivation gains one field and its core one result.
