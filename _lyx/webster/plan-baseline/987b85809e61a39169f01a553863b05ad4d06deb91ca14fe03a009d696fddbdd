# Card 9 — remove-orch-cleanup

**Delete:**
- `internal/orchcli/cleanup.go`
- `internal/orchcli#TestCleanup_Envelope`

**Edit:**
- `internal/orchcli#commandOf`
- `internal/cleanupcli#Command`
- `internal/cleanupcli#TestPrimeSegments`
- `internal/orchcli/doc.go`
- `internal/orchengine/doc.go`
- `docs/overview.md`

**Intent:**
D4: `lyx orch cleanup` goes, since `lyx cleanup` already removes old handoffs through its orch segment, which calls `orchengine.CleanupHandoffs` and `orchengine.CleanupRetiredNotices` in-process and is unchanged.

`commandOf` no longer adds `cleanupCmd`, which goes with `cleanup.go`, and `TestCleanup_Envelope`, the verb's own envelope test, goes with it; the engine functions stay covered by `TestCleanupHandoffs` and by `lyx cleanup`'s own tests.
`cleanupcli.Command`'s segment line for the orch no longer names `lyx orch cleanup` and says it is the orch's handoff cleanup; `TestPrimeSegments`' comment on a cycle in flight keeping every handoff no longer names it either.
`orchcli/doc.go`'s verb list and `orchengine/doc.go`'s handoffs and notices lines name `lyx cleanup --apply` instead; `docs/overview.md`'s orch and cleanup lines drop `lyx orch cleanup`.

**ImpactSummary:** removes one cobra verb with no caller outside its own command tree; the cleanup it ran stays reachable through `lyx cleanup`.

**Commit:** 9: Remove `lyx orch cleanup`; `lyx cleanup` removes old handoffs
