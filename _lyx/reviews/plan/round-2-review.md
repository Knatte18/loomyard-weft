---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/01-hub-workspace-file.md:146
    summary: Card 1 row 7 leaves open how the folder list changes, and a verb-driven change passes at card 1 but records no write once card 2 makes every verb regenerate
    fix: In 01-hub-workspace-file.md, TestRegenerateHubWorkspace row 7, replace "A changed folder list" with "A changed folder list, arranged by adding a pair with a raw git worktree add after a first call has written the file, as row 12 does, so no fabric verb regenerates". Add one sentence to the Tests list stating that a row expecting a call to write after the pair set changed changes the pair set with a raw git worktree add and never through hubforge.AddPair or another verb, because from card 2 on the verb regenerates by itself and the following call finds identical bytes. Text-only change; no target or verify change.
  - id: F2
    severity: LOW
    class: design
    location: _lyx/plan/02-verbs-regenerate.md:61
    summary: TestVerbsRegenerateHubWorkspace row 5 passes even when Add regenerates nothing on its rollback exit, so it cannot guard the D4 rollback requirement
    fix: In 02-verbs-regenerate.md row 5, remove the workspace file first, as rows 4 and 7 do, then run the Add failed by SetAddFailAfterWarpHookForTest and assert the file exists again, lists the prime and Bolt, and lacks the pair. With the file present beforehand, the file never lists a pair whose Add never completed, so the current assertion holds with no regeneration at all. Text-only change; no target or verify change.
  - id: F3
    severity: LOW
    class: scope
    location: _lyx/plan/04-retire-portals.md:126
    summary: Card 4 rewords every comment that names portal teardown except the doc of applyStalePairOwnership in prune.go, which still names portal debris
    fix: In 04-retire-portals.md add the Edit target internal/fabricengine#applyStalePairOwnership, and add a bullet to the comment-rewording list saying applyStalePairOwnership's doc, which says removeStalePair still has "portal, launcher, and worktree-registration debris" to clear, names launcher and worktree-registration debris only. Checked with lyx loom validate-plan --overlay on card 4 with the added target; the run reports no blocking finding beyond the baseline, and redundant-file-target does not fire because no other target names prune.go.
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/05-retired-artifact-sweep.md:81
    summary: Card 5 plants fabric-shaped portal links with os.Symlink on the premise that no verb creates them, which overlooks fslink.CreateDirLink and gives a Windows file symlink for the dangling link; card 4's launcher refusal row leaves the link type open
    fix: In 05-retired-artifact-sweep.md replace "plants portal links by hand with os.Symlink, as no verb creates them any more" with "plants portal links by hand with fslink.CreateDirLink, the call the retired portal creation used", and use that call for the fabric-shaped links and the link pointing elsewhere in rows 1 to 3 and in TestSweepRetiredHubArtifacts_UnresolvablePrime. CreateDirLink needs no existing target, so the dangling link of row 1 is a junction on Windows, the shape real hubs hold, where os.Symlink would need privilege and would make a file symlink for a missing target. Rows 4 and 5, which need a symlink at a file path and escaping symlinks, keep os.Symlink and skip on a permission error, as internal/standalonestate's symlink test does. In 04-retire-portals.md, in the Remove_Refuses... row, say the moved launcher directory's replacement is an fslink.CreateDirLink link to the directory outside the hub, as the replaced row's drifted link was.
---

### [MEDIUM] Card 1 row 7 breaks at card 2's gate if the folder list changes through a verb

**Location:** `_lyx/plan/01-hub-workspace-file.md:146`
**Issue:** Row 7 asserts that "a changed folder list" records `KindFileWritten` at the file's path after an unchanged call recorded none.
The card never says how the list changes, and every other row builds pairs with `hubforge.AddPair`.
At card 1 alone that works, because `Topology.add` in `internal/fabricengine/add.go` does not yet regenerate, so the explicit call finds a changed list and writes.
From card 2 on, `Add` regenerates on its own exit, so the file already lists the pair when the row calls `RegenerateHubWorkspace`, the call finds identical bytes and records nothing, and the card-2 gate, which reruns this integration test, fails.
Card 1 already knows the hazard in row 12 ("so no verb regenerates") and in its note that `CloneHub` writes the file from card 2 on, but not here.
The other card-1 assertions I traced hold with the repo as it stands after card 1 alone: the ide checks measure a task spawn around `Spawn`, `checkPrimeSettingsReadFailure` removes the file first through `resetPrimeEditorState`, and card 2 deletes the step that would see `Remove` rewrite the file.

### [LOW] Card 2 row 5 cannot fail without the rollback regeneration

**Location:** `_lyx/plan/02-verbs-regenerate.md:61`
**Issue:** Row 5 asserts that an `Add` failed by the hook "leaves the file without the pair".
No regeneration runs between the pair's creation and its rollback, so the file never lists that pair whether or not the rollback exit regenerates.
D4 and the acceptance criteria require the rollback exit to regenerate, and the Checkout rollback case in row 4 and the pre-flight rows in row 7 already prove their point by removing the file first.
Row 6 is sound because the file lists the pair before the `Remove` that fails.

### [LOW] One doc comment naming portal debris is outside every card

**Location:** `_lyx/plan/04-retire-portals.md:126`
**Issue:** D8 requires every doc comment that names `_portals` or its teardown to change.
Card 4 lists the comments to reword, and a search of the production sources for portal wording finds all of them in that list except the doc of `applyStalePairOwnership` in `internal/fabricengine/prune.go`, which says `removeStalePair` "still has portal, launcher, and worktree-registration debris to clear".
No card targets that symbol, so the comment would outlive `removePortal` and name teardown work that no longer exists.
The other portal wording I found outside the lists is historical narrative in `livestate_doc_test.go` and a comment inside `TestGate_ZeroValueDeclarationsAreRefusals`, which card 4 does list, so neither is raised.

### [LOW] Card 5 plants retired portal links with os.Symlink against the repo's link rule

**Location:** `_lyx/plan/05-retired-artifact-sweep.md:81`
**Issue:** CLAUDE.md routes every link through `internal/fslink` and says never to rely on Windows file symlinks.
Card 5 plants "portal links by hand with `os.Symlink`, as no verb creates them any more", but `fslink.CreateDirLink` is the call `createPortal` used and survives card 4.
On Windows the dangling link of row 1 becomes a file symlink when the target is missing and needs symlink privilege, while real hubs hold junctions there, so the rows exercise a shape the sweep never meets in production.
`TestSweepRetiredHubArtifacts_UnresolvablePrime` is a unit test with no stated way to create its link, and the repo's unit tier skips on Windows where `os.Symlink` is needed.
Card 3 already chose `fslink.CreateDirLink` for the same reason.
Card 4's launcher refusal row says "a symlink to it is left in its place", where the row it replaces used `fslink.CreateDirLink` for its drifted link.

## Rejected

- Focus 1, ideengine steps through `hubforge.AddPair`: not a defect.
  `checkTaskSlugOpensBareFolder` compares the file around the task `Spawn`, `checkPrimeSettingsReadFailure` and `checkPrimeWriteFailure` start from a removed file, and card 2 deletes the survive-topology step, so no earlier assertion fails.
  The one hazard in card 1 is row 7, filed as F1.
- Focus 2, hub containment: not a defect.
  `lyxcwd.ResolveWorktree` succeeds for a worktree registered outside the hub, because its hub path is the parent of that worktree and the recorded anchor falls back to `.`.
  The outside entry therefore reaches the direct-child skip rather than the resolve-failure skip, and row 5's third bullet exercises it.
- Focus 3, destruction chokepoint: not a defect.
  The sweep passes `checkPathRequest` before every removal, `removeLink` runs it internally, `resolvePathOwnership` is edited to admit `_portals` beside `_launchers`, and every refusal or error becomes a `HubDetails` entry.
  Calling `removeContainedPath` from `portals.go` and appending `KindPathRemoved` there follows `removeLaunchers`, and the guard in `cmd/lyx/destructiveguard_test.go` bans only the primitive tokens, so no finding is raised.
- Focus 4, verify coverage: not a defect.
  Every package a card targets is in the plan's `## verify:` lines.
  The docs link scan lives in `internal/lyxcwd`, which no card targets.
- Focus 5, reachability: not a defect.
  Rows 9 to 11 of card 1 and rows 7 and 8 of card 2 reach their branches.
  Row 12 of card 1 fails both without the lock and with the listing moved before it, because the hook sits just before the lock and the pair is created after the hook's signal.
  Card 5's escaping-`_launchers` row is refused at ownership on a root anchor and at containment on a nested one.
- Card 2 registers `Remove`'s regeneration before `lockPairTeardown`, so a teardown-lock timeout also regenerates, though D4 says a pre-flight refusal regenerates nothing.
  The exit is after `RemoveRefusal` and `inspectPair`, the bytes stay identical unless the file was stale, and the placement is what lets the regeneration run after the lock is released; not raised.
