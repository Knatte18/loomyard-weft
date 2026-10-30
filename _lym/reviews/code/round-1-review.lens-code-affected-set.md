```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, covering what the diff's changes force outside the lines it touches.

**Per change, the set found and whether it is covered**

- **`ResolvePullRequest` gets a fifth parameter.**
  - The only non-test callers are `prreview.go:89` and `finalize_pr.go:125`, and both pass `hasCommits`.
  - Test callers are outside this lens.
  - Covered.
- **New seam `PullRequestDeps.TaskHasCommits`.**
  - Set: the `deps.go` field, `loomcli/landingdeps.go:337`, the `envkit` `LandingDeps` fixture, the PR-Review `checkWiring` nil check, Finalize's use, and the two wiring-seam allowlists.
  - The allowlists are `loomcli/wiring_seams_test.go:44` and `battencli/wire_seams_test.go:101`.
  - Finalize skips the whole step when `ParentMode` is nil. `battencli` is declared loom-only and lands no branch, so the missing nil check in Finalize is unreachable there.
  - Covered.
- **`githubHaltReason` text split and `tokenFailure` ordering.**
  - `github`-call sites in `landingshed` that classify a failure all go through `githubFailure` or `pollFailure`, and both were changed.
  - The remaining `ErrTokenUnresolvable`-only checks outside `landingshed` are `hubissues.go:197` and `selfreport.go:109`.
  - Both are outside the D5 scope, which names PR-Review and Finalize only, so I raise nothing.
  - Covered.
- **`ErrTokenRejected` and `StatusCode`.**
  - The set is the `transport.go` wrap, the `token.go` declaration, and the `transient.go` and `doc.go` text.
  - Covered.
  - `docs/overview.md:410` describes token resolution only and does not need an edit.
- **Docs and PATTERN.**
  - The `PATTERN.md` `PATTERN-pull-request-mode` text matches the declared text character for character.
  - That entry has no background file in the `pattern/` link list, so there is none to update.
  - The `landingshed/doc.go` and `githubclient/doc.go` edits describe the new behavior.
  - A grep of `*.md`, `*.tmpl` and `*.yaml` for `GH_TOKEN` found no stencil or spec that restates the old token-for-every-failure halt text, so nothing is left stale.
  - Covered.
- **`PATTERN-refusal-way-forward`.**
  - The 422 and other-failure halts each name a way forward (`lym loom resume`), and the diff adds test rows for them.
  - The tests themselves are outside this lens.
  - Covered.
- **Commit messages.**
  - There is no `Pattern-Departure:` trailer and none is needed, since no PATTERN entry is departed from.
  - Covered.

**Findings:** none.

**Tool calls**
1. One combined grep: non-test callers of `ResolvePullRequest`, other `ErrTokenUnresolvable` sites, files mentioning `githubHaltReason` or `CommitsMissing`, docs mentioning `pull-request-mode`, `GH_TOKEN` in markdown, templates and yaml, and the wiring-seam allowlists. My context lacked the full caller and doc sets.
