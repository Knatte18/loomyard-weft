# Round 3 fixer-report — plan

## Changes

- **F1** (BLOCKING), fixed as proposed.
  Card 10's **Edit:** list gains `internal/shortcutcli/shortcuts.yaml`, `internal/shortcutcli#TestMount_File` and `cmd/lyx#TestShortcuts_FileAndRootListing`.
  A new Intent bullet says the shipped `prime` shortcut's first argv moves from `orch start` to `beam start`, which exists until card 11 reworks it, and that `TestMount_File`'s `prime` row and the operator-index line read `shortcut: lyx beam start, then lyx reed attach`.
  Card 12's Intent now says `s` replaces the `prime` that card 10 already repointed, where it said `prime` named the removed `lyx orch start`.
- **F2** (BLOCKING), fixed as proposed.
  Card 15's last Tests paragraph now says `TestMain` runs from its own untagged support file, every test file carries the `tier0` tag, and every test calls `t.Parallel`.
- **F3** (MEDIUM), fixed as proposed.
  Card 16's **Edit:** list gains `internal/testkit/envkit#FullEnv`, `internal/testkit/envkit#nilLegal`, `internal/loomcli#intentionallyNil` and `internal/battencli#intentionallyNil` after `internal/shedrecipe#Env`.
  Its Intent gains the sentence on `FullEnv` filling the `Pawl` seams, `nilLegal` listing `Pawl.Watch`, and both `intentionallyNil` tables listing the remaining `Pawl.<Seam>` paths.
- **F4** (MEDIUM), fixed as proposed.
  Card 14's **Edit:** list gains `internal/testkit/envkit#nilLegal` and `internal/loomcli#TestLandingDeps_EveryFieldPopulated` after `internal/landingshed#Deps`.
  Its `Deps` bullet gains the `nilLegal` entry `Landing.LandBlockers` and the `unfilled` map's `LandBlockers` and `ParentWorktree`.
- **F6** (LOW), fixed as proposed.
  Card 10's **Delete:** list gains `internal/orchcli#liveFixture.startPlainRun`, `internal/orchcli#killTreeProcs` and `internal/orchcli#tailString` after `TestSmokeOrch_Adopt`.
  The Tests paragraph says they go with the adopt smoke while `waitForDiag` stays for `TestSmokeOrch_CompactCycle`.
- **F7** (LOW), fixed as proposed.
  Card 16's Tests paragraph and its two **Edit:** glyphs name `TestNewPawl` and `TestPawlRouting`.

`lyx loom validate-plan` returns `ok: true` with informational findings only.

## Self-check

- fix-claims, LOW: card 16's `Pawl.Watch` `nilLegal` row is dead if `Watch` is channel-typed, since `walkNil` checks no channel.
  Fixed: card 16's sentence now calls `Pawl.Watch` a function seam, as card 15's `Deps` bullet makes every seam ("each a function the producers check is non-nil").
- fix-proposal: no findings.
- fix-consistency: no findings.
  It also noted, outside its lens, that between cards 10 and 11 `lyx prime` runs the pre-card-11 `beam start`, which starts dobbies but no orch, before attaching.
  No change: the shortcut resolves and passes its gate at every card, and card 11 makes `beam start` bring up the orch two commits later within the same task.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
