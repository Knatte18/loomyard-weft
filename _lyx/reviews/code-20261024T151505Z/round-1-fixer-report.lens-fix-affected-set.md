```yaml
lens: fix-affected-set
outcome: reported
```

No findings.

- **F1** (role compared to the orch role exactly): I searched for every non-test `Role ==` or `Role !=` against `RoleOrch` in `internal`. The only hits are `internal/tenderengine/report.go:219`, `internal/hibernateengine/verbs.go:19`, `internal/orchengine/pause.go:149` and `internal/orchengine/pause.go:190`, plus `internal/tendercli/cli.go:46`. All of them also require `Slug == ""`, so they select the hub orch, and an exact match is correct there. `isRunAgentSession` was the only site that needed `MatchesRole`.
- **F2** (rows in the test table): the new `ly:some-task:orch-2` and `ly:webster` rows are in the one table, and nothing else tests the guard.
- **F3** (docs for the refusal): the only plugin or doc files that mention the note way forward or claimed entries are `plugins/ly/skills/board/SKILL.md`, which the fix updated, and `internal/boardengine/doc.go`, which the review says the diff already covers.
- **F4** (doc comment wording): this is a single site with no set.

I left no finding unchecked.

Tool calls:
1. `grep` for `RoleOrch` comparisons in non-test Go files, to check every site that tests the role.
2. `grep` over plugins and docs for the note way forward and claimed entries, to check every doc site.
