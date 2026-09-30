# Fixer report — plan round 1

All nine findings are fixed in the plan files.
`approved:` in `00-overview.md` is untouched.
`lyx loom validate-plan` returns `ok: true` after the edits, with only three informational `create-new-unit` notes.

## Changes

- **F1** (`03-add-remote-preflight.md`): the unarchived-leftover test now also asserts that no local `<slug>-weft` branch exists after the refusal.
- **F2** (`03-add-remote-preflight.md`): the SkipPush test now asserts that origin's warp branch is unchanged as well as its weft branch, and says why the warp assertion holds.
  The Intent now states that step 11's warp push is not gated by `SkipPush`/`SkipGit` and that changing it is outside this plan's scope.
- **F3** (`03-add-remote-preflight.md`): the ImpactSummary is rewritten.
  It now names the three zero-`AddOptions` entry points (`fabriccli`'s `runAdd`, `battencli`'s create-worktree row, `hubforge.AddPair`), the two added `ls-remote` reads, the earlier failure point for an unreachable origin, and that no existing caller re-adds a removed slug.
- **F4** (`03-add-remote-preflight.md`, `05-add-replace-archived-weft.md`): the warp fast-forwardable test moved from Card 5 to Card 3.
  It now uses `Remove(l, slug, false, true)`, so the remote weft branch is gone and Card 3 can reach success on its own.
- **F5** (`05-add-replace-archived-weft.md`): the ImpactSummary now covers the post-replacement rollback outcome and the new `addBeforeWeftReplaceHook` var.
- **F6** (`02-gate-archived-weft-delete.md`): added an integration case in which a valid archive tag with an empty lease SHA is refused on dirtiness and origin keeps the branch.
- **F7** (`03-add-remote-preflight.md`): the ancestor helper now wraps `gitrepo.New(dir).IsAncestor` after its missing-object pre-check, and `internal/gitrepo#Repo.IsAncestor` (copied from `lyx quarry glyphs`) is added to `Uses`.
- **F8** (`02-gate-archived-weft-delete.md`): the card now says to update `destroyremote_integration_test.go`'s file header to describe the new executor-level cases.
- **F9** (`03-add-remote-preflight.md`): "the next card" now reads "Card 5".

## Deferred

None.
