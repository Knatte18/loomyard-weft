```yaml
lens: fix-claims
outcome: reported
```

Task: check that every claim the fix added or rewrote holds against the code at HEAD.

No findings.

The claims checked and found to hold:
- **D1 bound:** `internal/fabriccli/anchor.go:468` calls `EndDobbies(..., runproc.RolePush, runproc.RoleSync)`. `dobbyOwner` maps the prime to the hub owner (`anchordeps.go:91`), and `EndSparing` ends every role not spared (`runproc/end.go:64`). A worker registered under `HubOwner` is therefore ended.
- **D10 bound:** `orchcli` `prepare` only appends a warning when the origin is unreachable (`start.go:446-447`). The owner refusal comes later, in `beamUpRefusal(..., facts.owner, ...)` (`start.go:498`). So a stale local Bolt that reads this machine as owner lets the beam start.
- **D11 rationale, teardown precedent:** in `pairteardown/skip.go:34`, an `owned_elsewhere` verdict only skips the quiet wait and driver checks. The removal then reaches fabric's tag-and-push archive (`archive.go:1-4`, `remoteteardown.go:127`), and nothing there branches on the verdict.
- **D11 rationale, `archive-owner/` prefix:** `ArchiveTagSlug` cuts `ArchiveTagPrefix = "archive/"` (`archive.go:37-50`), so `archive-owner/...` does not parse. The run, cost, timing and raddle readers all go through `ArchiveTagSlug`. `remoteArchiveTags` lists `refs/tags/archive/<slug>/*` (`remoteleftover.go:224`). The new prefix stays out of both.
- **D11 rationale, rule text:** `PATTERN-owner-machine` quotes "Run start, resume, push or landing: only if no other machine owns its entry". Its push bullet is "A loom run's commit and push of its pair".
- **D11 bound, failed owner pause:** `owned.go` logs "the next probe tick runs it a full pass", and `planPass` gives `full` on `ProbeTick && !covered` (`passes.go:124-127`).
- **D12:** `--drop-local` tags through `keepArchiveTag` → `ArchiveTagName`, which uses the `archive/` prefix (`destroy.go:1933`). Teardown uses the same name. So `--archives` lists none of them.

Tool calls made:
1. Grepped `internal/orchcli/start.go` for owner/unreachable handling and `runproc` for `EndSparing`, to check the D10 and D1 bounds; my context had only lines 440-450 of `start.go`.
2. Grepped `internal/pairteardown` for its archive and verdict handling, to check the teardown precedent in D11; my context had only `doc.go` lines.
3. Grepped `fabricengine` and `pairteardown` for `EntryOwnedElsewhere` and the archive call sites, to confirm the teardown archive does not branch on the verdict.
