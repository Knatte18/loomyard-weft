---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/boardcli/verbs_test.go:805
    summary: the beam-orch refusal table has no row for a numbered orch role or a non-string project in the payload, two branches of the new guard
    fix: In the "a beam's orch changes the entries of its own beam only" subtest of stepRunSettings, add one passing upsert as strand ly:bm:orch-2 on an entry of the beam (for example {"slug":"member","profile":"speed"}) so the MatchesRole numbered-role branch is pinned, and add one refusals row with strand ly:bm:orch and payload {"slug":"member","project":5,"profile":"docs"} so a non-string project is pinned as refused.
---

### [LOW] Two branches of the new guard have no pinning row

**Location:** internal/boardcli/verbs_test.go:805
**Issue:** `refuseRunSettingsCaller` admits a numbered orch role through `agentname.MatchesRole`, and it reads a non-string payload `project` as the prime link and refuses.
The new subtest uses only `ly:bm:orch`, `ly:bm:loom` and `ly:other:orch`, so changing `MatchesRole` back to a plain equality or letting a non-string `project` through would pass every row.
Both are behaviours the diff adds; the fix is a row in the existing table, not a new test.

## Rejected

- Lens code-tests finding on the `BeamUnder` error path and the engine test not setting `Config.ParentOf`: the boardcli subtest runs the real CLI wiring, which sets `cfg.ParentOf` to `hubgeom.ParentOf` (`internal/boardcli/cli.go:146`), and its `upsert deep` row passes through the group link on that seam path; the error paths only propagate a read error, which the old code did not test either.
- The orch role text is unchanged: `contracts/orchrole/orch-role.md:218` already tells every orch, not only the hub orch, to name a profile in the entry's `profile` field before the batten run starts, so the Goal's first sentence needs no diff.
- `internal/configcli/hubwide.go` still refuses on `agentname.TaskSlug`: it guards hub-wide config writes, a separate surface the Goal does not mention.
- `BeamUnder` and the guard against `ParentBeam`, `hubgeom.ParentOf` and `PATTERN-owner-machine`/`PATTERN-pair-parent`: the guard reads a project link through the one resolver seam and picks no pair parent or owner, so no PATTERN entry is touched.
- Unparseable strand names passing the guard: the old `TaskSlug` guard let them through too, so the behaviour is unchanged.
