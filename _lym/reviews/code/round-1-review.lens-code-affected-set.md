```yaml
lens: code-affected-set
outcome: reported
```

**Lens code-affected-set.** I found no finding. Each change below was checked against the callers, readers, tests and docs I could reach.

- `Deps.UnderAdopt`, `PullBolt` and `ArchiveOwner` (dobbyengine): covered. Every field is nil-safe, and `loop_test.go` and `dobbytrace_integration_test.go` build `Deps` without them. `hubDobbyDeps` is the one production constructor, and it fills all three. `internal/dobbyengine/doc.go` and `internal/beamcli/doc.go` describe them.
- `OwnerWatch.UnderAdopt` (orchengine): covered. It is nil-safe, `internal/orchcli/watch.go` is the one production wiring, and `watcher_owner_test.go` gained a row. `internal/orchengine/doc.go` is updated.
- `BeamUpOptions.ArchiveOwned` (orchcli): covered. The one production setter is `beamcli/cli.go`, nil means no archive, and `internal/orchcli/doc.go` and `internal/beamcli/doc.go` are updated. `lym adopt`'s `BeamUp` callers leave it nil, and an archive there would be unwanted anyway.
- The `archive-owner/` prefix against readers of `archive/` tags: covered.
  - `ArchiveTagSlug`, `ArchiveTagPrefix`, `recordsreads.go:56`, `remoteleftover.go:224` and `cleanupremotewarp.go:239` match only on `archive/`.
  - The reportengine readers (`raddle.go:71`, `runs.go:139`, `cost.go:136`, `timing.go:242`, `source.go:77`) go through `ArchiveTagSlug` or `ArchiveTagPrefix`, so none of them lists a D11 tag.
  - `TestOwnerArchiveTagName` pins the name and the `ArchiveTagSlug` miss.
- `tagAndPushArchiveTip` and `createArchiveTag` split into `pushArchiveTag` and `createTagAt` (fabricengine): covered. Behavior is unchanged and the teardown archive tests still reach both.
- `fabricengine/doc.go`: covered. It documents `ArchiveOwnerTips`.
- Repo rule documents, `PATTERN.md` and `pattern/`: nothing to change.
  - The diff touches neither, and the directive declares no PATTERN entry to land.
  - `PATTERN-owner-machine` and `PATTERN-pair-remote` stay consistent with D11: the archive pushes tags only, it holds the teardown lock, and the git calls stay in `fabricengine`.
  - No pattern file or spec names the dobby's foreign-owner exit, and `docs/overview.md` has nothing on it.
- Commit messages: none carries a `Pattern-Departure:` trailer, and none is needed.

Notes outside this lens, for the handler:
- `pauseRunsForOwner` (`handover.go:99`) skips a run already marked `OwnedElsewhere`, so a failed owner pause is not retried on a later archive retry. That matches `owned.go`'s "never retried blind" and D11's bound, and I raise no finding on it.

Tool calls:
1. Grep for `Deps{`/`OwnerWatch{` constructors and the exit message across the repo, to find every constructor and text reader.
2. Read `internal/dobbyengine/owned.go`, because `ownedElsewhere`'s behavior on a repeated hand-over was not in context.
3. Grep of md/yaml/tmpl for prose about the dobby's exit on a foreign owner. It found none.
4. Grep of `pattern`, `PATTERN.md`, `CLAUDE.md` and the specs for `archive/<slug>`, `archive-owner` and the owner-exit prose, plus the PATTERN diff stat, to check the rule documents.
5. Grep of `docs/overview.md` for dobby, watcher and adopt-record passages. It found no passage the diff makes wrong.
