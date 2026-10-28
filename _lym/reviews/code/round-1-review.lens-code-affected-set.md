```yaml
lens: code-affected-set
outcome: reported
```

Lens: code-affected-set — what the diff's changes force outside the lines it touches.

**Per change: the set found, covered or not**

- **`vscode.WriteConfig`, `Look`, `PickColor` removed, `SeedTask` renamed `Seed`:** the callers are `ideengine/spawn.go` and the tests. No stale caller remains. Covered.
- **`slugVerbCommand` signature:** the callers are the `seed` and `open` wiring in `idecli/cli.go`. Covered.
- **Hub workspace splice (`beamWorkspaceSettings`):** the set is `fabricengine/doc.go` and the `hubworkspace` tests. The tests are added, but the doc is not updated. See finding 1.
- **`beam_color` Task field:**
  - `store.go`, `replay.go`, `migrate.go`, `owners.go`, `task.go` and `boardengine/doc.go` are covered.
  - The upsert key list and the `boardcli` payload help are not covered. See finding 2.
- **`seed_ide_command` / `template.yaml`:** the wording is aligned with `docs/overview.md`. This is a doc-truthing edit and matches the entry's "ide seed writes colours" scope. Covered.
- **`OwnerRequest.BeamColor`:** the other callers (`orchcli`, `battencli`, `adoptcli`, `hubgeom`) leave it nil, which is a valid request. Covered.
- **Docs:** `docs/overview.md` is updated.
  - `ideengine` and `idecli` have no `doc.go`; their file headers were updated.
  - The `vscode` package doc in `config.go` was updated.
  - `beamcli` has no package doc touching this.
  - `fabricengine/doc.go` is the one gap, finding 1.

**Findings**

1. **MEDIUM, `consistency`, `internal/fabricengine/doc.go:1131-1133`.**
   - `hubworkspace.go` now splices a beam's own `.vscode/settings.json` into that beam's `.code-workspace` (D8, `hubworkspace.go:62-77`).
   - The module doc still says the beam file lists "the beam, its parent, `Bolt` and every pair whose parent is the beam".
   - It also says "settings belong in the prime's `.vscode/settings.json`, which the file splices verbatim".
   - The second statement is now false for a beam file, and `fabricengine/doc.go` is not in the diff stat.
   - The decision record's scope and CLAUDE.md's "Docs land in the same commit" require the touched package docs to land.

2. **LOW, `consistency`, `internal/boardengine/store.go:456-458`.**
   - `UpsertPayloadKeys` and its comment name `owner_machine` and `beam` as keys accepted for round-tripping, but `beam_color` is not listed.
   - `validateUpsertFields` (`store.go:472-474`) rejects any key outside that list as unknown.
   - A `lym board get` payload on a beam now carries `beam_color`, so a read-edit-upsert of that payload is refused.
   - D1 makes it "no upsert key" on purpose. The affected set is the `lym board` upsert help, `boardcli/cli.go:500-508`, and the `UpsertPayloadKeys` comment.
   - Neither says that `beam_color` is output-only. If the refusal is intended, the refusal and the help should say so; if round-trip is intended, the key belongs in the list as an accepted no-op like `owner_machine`.

**Tool calls**
1. Grepped for callers and doc mentions of the changed symbols. My context lacked a repo-wide search.
2. Listed the `doc.go` files and grepped them for ide and settings mentions. My context lacked which package docs exist.
3. Read `fabricengine/doc.go:1125-1136` and grepped for `beam` and `owner_machine` field lists. My context lacked the doc text and the upsert key sites.
4. Read `boardcli/cli.go:500-514` and `store.go:450-462`. My context lacked the payload help and the upsert key list.
5. Read `store.go:459-482`. My context lacked how an unknown upsert key is handled.
