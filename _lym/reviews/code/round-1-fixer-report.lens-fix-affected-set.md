```yaml
lens: fix-affected-set
outcome: reported
```

Lens fix-affected-set. I ran one search per finding and found nothing the fix missed.

No findings.

Unchecked: none. All three findings have a set premise and each got one search.

- **F3 (stale "verbatim" claim):** `grep -i verbatim` over `internal/vscode`, `internal/fabricengine/doc.go`, `internal/fabricengine/hubworkspace.go`, `internal/ideengine`, `docs`, `PATTERN.md` and `pattern` matched settings or workspace text in two places.
  - `internal/vscode/workspace.go:24` says `BuildWorkspace` splices settings verbatim. That is true of what it receives, since it embeds whatever bytes the caller passes, so it needs no change.
  - `internal/vscode/workspace_test.go:30` is a test row name and needs no change.
  - The Limitation text in `internal/vscode/config.go` was the only stale claim, and it is now reworded.
- **F2 (no-shortname title):** The new subtest covers both title sites the fallback serves, the hub file (`l.RepoName`) and the beam file (slug). No other test exercises the no-shortname branch of `workspaceWindowTitle`, so this is the only pin.
- **F1 (Seed regeneration error):** In `internal/ideengine`, `Seed(` is called only in `spawn_hub_integration_test.go`, once in `checkPrimeSeedWritesHubWorkspace`, once in `checkTaskSlugOpensBareFolder` and now once in `checkPrimeWriteFailure`. The prime-seed error return was untested before, and the new assertion in `checkPrimeWriteFailure` covers it.

Tool calls:
1. `grep -i verbatim` over the docs and package paths, to find every sibling that makes the "spliced verbatim" claim.
2. `grep BuildWorkspace|Seed(` over `internal`, to find callers and tests of `Seed`. The context held only the changed test files. The pattern matched mostly unrelated seed helpers, so it added little.
3. `grep` over the fabricengine and ideengine tests for workspace title assertions, to check no other test already pins the no-shortname fallback.
