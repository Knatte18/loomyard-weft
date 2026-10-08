# Plan-Review round 1 — fixer report

Every finding was validated against the code and the decision record, and all five are fixed.
`lyx loom validate-plan` passes after the fixes (`ok: true`); its only findings are the informational `create-new-unit` rows for `runlock_test.go` and the new kit.

## Fixed

- **F1 (BLOCKING, Card 3 copies a shared fixture).**
  Confirmed: `markDoneFixtureDir` in `internal/loomcli/landingdeps_markdone_test.go` is the only test that writes `boardengine.ConfigTemplate()` into the hub's board config, and Card 3 told the implementer to write the same thing in battencli.
  One detail in the review's premise is wrong, but the violation stands: `internal/ideengine/menu_test.go`'s `writeBoard` also writes a board config, though a hand-written one, not the template.
  Fix: Card 2 now creates `plan:internal/testkit/boardkit#HubWithBoardConfig` -> `func HubWithBoardConfig(t *testing.T) string` in a new kit, and lists `internal/loomcli/landingdeps_markdone_test.go` under `Edit:` so `markDoneFixtureDir` builds its hub through the kit.
  Card 3 lists the kit under `Uses:` and builds `TestWire_CreateWorktreeClaimsTheBoardEntry` on it.
  Card 2's Intent states the kit's import bounds (`boardengine`, `configengine`, `fabricengine`, standard library; no `internal/*cli`, no `os/exec`), and its ImpactSummary now covers the fixture move.
  The kit is bundled into Card 2 rather than given a card of its own.
  It is a helper serving the `OpenHub` surface's tests, since a hub with a board config is what `OpenHub` opens.
  Inserting a card before Card 3 would also mean renumbering `03-batten-claim.md`, and this round can only write files, not rename or delete them, so the old file would be left behind for `index-file-mismatch` to flag.
- **F2 (LOW, Card 3 re-pins `SetRunStatus`'s skip).**
  Dropped the `done` and absent-slug rows from `TestWire_CreateWorktreeClaimsTheBoardEntry`.
  The no-status row and the no-board-config row stay, and the card says the skips are pinned by Card 2's `TestSetRunStatus`.
- **F3 (LOW, Card 1's every-write rows on unlocked entries).**
  Replaced "every write on an entry with no, empty, `done`, `abandoned` or `active` status" with `IsRunStatus` rows for those five values plus one body-changing upsert on an `active` entry, which shows the check consults the predicate.
- **F4 (LOW, no test of the claim surviving a refused `Topology.Add`).**
  Confirmed: Card 3's unit test fails at `fabricengine.LoadConfig`, before `Add`.
  Card 3 now lists `internal/battencli/lifecycle_integration_test.go` under `Edit:`.
  The card extends `stepCreateRow_LeftoverBranchIsRewordedForPrime`, which already reaches an `Add` refusal over a real hub: the step seeds the entry with the file's existing `seedBoardTask` and asserts `running · Worktree-Create` after the refusal.
  No top-level test is added.
- **F5 (NIT, incomplete list of mutating Board methods).**
  Dropped the parenthetical from Card 1's `boardCriticalSection` bullet.

## Disputed

None.

## Deferred

None.
