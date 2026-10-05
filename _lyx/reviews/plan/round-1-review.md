---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/04-retire-worktree-copy.md
    summary: Card 4 stops clone materializing a per-worktree board.yaml before card 5 moves the board readers, so hubforge-hub tests that still read the worktree copy fail at card 4's commit
  - id: F2
    severity: MEDIUM
    class: scope
    location: _lyx/plan/03-hub-wide-reconcile.md
    summary: Card 3 seeds every hub-wide module, fabric included, from the prime's copy and warns about custom types and labels for fabric; D4 licenses the prime seed for board.yaml only, and the legacy fold-in precedence and its Seed value are unstated
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/plan/07-config-cli-hub-wide.md
    summary: The hub-wide write validates after writing and restores in-process, but the Intent states no bound for a failed restore or a killed process, which leaves an invalid file for the next whole-repo board commit
  - id: F4
    severity: LOW
    class: design
    location: _lyx/plan/07-config-cli-hub-wide.md
    summary: The concurrent-edit refusal of the staged editor has no entry in the Messages list and no coverage row
  - id: F5
    severity: LOW
    class: consistency
    location: _lyx/plan/04-retire-worktree-copy.md
    summary: Card 4 adds Retired and Divergent but names no configsync doc update, and card 3 says the configsync.go header points at ReconcileFabricAt although only ReconcileAll's doc and the fabric.go comments do
  - id: F6
    severity: LOW
    class: scope
    location: _lyx/plan/05-board-readers-hub-file.md
    summary: plugins/ly/skills/board/SKILL.md still places board.yaml at the worktree's _lyx/config, and docs/shared-libs/configengine.md keeps board.yaml as its generic per-worktree example path
---

### [MEDIUM] Card 4 precedes card 5 and breaks hubforge-hub tests at its own commit

**Location:** `_lyx/plan/04-retire-worktree-copy.md`, `_lyx/plan/05-board-readers-hub-file.md`
**Issue:** Card 4 changes `CloneAndWire` so `ReconcileAll` skips `board`, and clone therefore no longer materializes `_lyx/config/board.yaml` in the weft.
Until card 5 lands, `boardcli`, `openHubBoard` and `ideengine.Menu` still load the per-worktree file.
`internal/loomcli/landingdeps_integration_test.go:51` builds a `hubforge` hub and calls `boardengine.LoadConfig(location.AnchorPath(), "board")`; it reads exactly the file card 4 stops writing, so it fails at card 4's commit.
Card 5 itself says that test is fixed in card 5.
The two cards share no symbol, so the spec's derived order (targets and `Uses`) does not force card 5 first; plan order breaks the tie and puts card 4 first.
**Fix:** Reorder so the readers card runs before the retire card: renumber the files and the Card Index so "board-readers-hub-file" is card 4 and "retire-worktree-copy" is card 5.
Card 5 depends only on card 3 (the hub file from clone reconcile), so it passes with per-worktree materialization still in place.

### [MEDIUM] Hub-wide seeding is wider than D4

**Location:** `_lyx/plan/03-hub-wide-reconcile.md`
**Issue:** D4 licenses seeding the absent hub `board.yaml` from the prime's copy, and keeps `fabric.yaml`'s legacy warp/weft fold-in.
The card seeds every hub-wide module from `configengine.ConfigFile(primeBaseDir, name)`, so an absent hub `fabric.yaml` is seeded from a per-worktree `fabric.yaml`.
D5 records that such a file is one `lyx config fabric` edited and nothing ever read, so seeding from it imports values (`pathspec`, `branch_prefix`) that were never in effect.
The card also leaves three things open:
- whether the prime copy or the legacy `warp.yaml`/`weft.yaml` fold-in wins when both exist;
- which `Seed` value a legacy fold-in reports, since `Seed` is `"hub"`, `"prime"` or `"template"` only;
- the template-seed warning says custom types and labels were not carried, which is wrong for `fabric`, and the existing heal test deletes the hub `fabric.yaml`, so that warning fires for it.
**Fix:** Restrict the prime seed to modules that declare open maps (`board`), or state in the Intent why seeding `fabric` from a never-read file is wanted.
State the legacy fold-in as the only fabric source besides the template, report it under a stated `Seed` value, and scope the template warning to modules with open maps.
Add the fabric rows to the table-driven test accordingly.

### [MEDIUM] Write-then-validate has no stated bound for a failed restore

**Location:** `_lyx/plan/07-config-cli-hub-wide.md`
**Issue:** The `--set` path runs `configengine.Set` on the hub file, validates with the strict load, and restores the snapshot on failure; the editor path does the same after the atomic copy.
D5 bounds a malformed edit to "a refused write, so it never reaches `_board`".
If the restore itself fails, or the process dies between write and restore, the invalid file sits in `_board`'s working tree, and the next board sync's `Bolt.Commit` stages everything and commits it.
The Intent names the guard (strict load) but not this window, and the rubric requires each guard's bound to be stated.
**Fix:** State in the Intent that a restore error is joined into the returned error naming the file as possibly invalid, that `--set` writes through `fsx.AtomicWriteBytes` so a kill cannot leave a torn file, and that a kill between a valid-looking write and validation is the residual window.
Alternatively validate the new bytes before they replace the hub file.

### [LOW] Concurrent-edit refusal has no message and no test

**Location:** `_lyx/plan/07-config-cli-hub-wide.md`
**Issue:** The record does not mention a concurrent-edit refusal; it follows from the plan's staging design, which is a licensed way to keep the lock off the editor, so the scope is acceptable.
But it is a new refusal path: the Messages list has no wording for it, and no coverage row reaches it.
**Fix:** Add its message (naming the hub path and saying it changed while the editor was open, with the staged edit left or discarded as the card decides) and one row in the table-driven `dispatch` test that changes the hub file between staging and the locked step.

### [LOW] Card 4 and card 3 doc statements disagree with the code

**Location:** `_lyx/plan/04-retire-worktree-copy.md`, `_lyx/plan/03-hub-wide-reconcile.md`
**Issue:** The decision record lists the `configsync` package doc among the docs that land in the same commit.
`configsync.go` has no `doc.go`; its header and the `Result` and `ReconcileAll` comments are the package doc.
Card 4 introduces `Result.Retired`, `Result.Divergent` and `ReconcileAll`'s `boardDir` parameter but says nothing about updating them.
Card 3 says the header "points at `ReconcileFabricAt`"; the header does not, while `ReconcileAll`'s doc and the comments in `fabriccli/fabric.go` (`runReconcile`) and `hubforge/seed.go` do.
**Fix:** Card 4 states that `ReconcileAll`'s doc and the `Result` field comments describe the retire and divergence outcomes.
Card 3 names `ReconcileAll`'s doc and the `runReconcile` comments, not the header.

### [LOW] Two docs still place board.yaml per worktree

**Location:** `_lyx/plan/05-board-readers-hub-file.md`
**Issue:** `plugins/ly/skills/board/SKILL.md:27` tells agents the label maps live in `_lyx/config/board.yaml`, which no longer holds after this plan.
`docs/shared-libs/configengine.md` lines 36 and 132 use `_lyx/config/board.yaml` as the generic `ConfigFile` example; card 6 fixes only the directory tree.
**Fix:** Card 5 reworks the SKILL.md sentence to name the hub file (`lyx config board` remains the way to edit it).
Card 6 changes the two examples in `configengine.md` to a per-worktree module such as `loom.yaml`.

## Focus items settled without a finding

- Docs line: card 1 (`configreg.go` header), card 3 (`configsync` comments), card 5 (`boardcli`, `boardengine` docs), card 6 (`configengine.md`) and card 7 (`configcli/doc.go`, `docs/overview.md`) cover the listed docs; the only gap is F5.
- D4 clone deviation: keeping `CloneAndWire`'s whole-repo `Bolt.Commit` is sound, because a fresh clone has no concurrent `_board` writer, and the existing push (`b.Push`, fatal on error) is carried unchanged; not a defect.
- D5/D6 scope: card 4's test rewrites follow from `ReconcileAll` no longer creating `board.yaml`, so each is required by D6; card 7's snapshot restore is required by "the hub file is left as it was"; the concurrent-edit refusal is covered by F4.
- Test economy: card 4's integration assertion covers the `retired` JSON key that `configcli.runReconcile` emits, which no untagged test reaches (`reconcile_test.go` holds one not-a-repo test); card 7's integration rows cover `resolveReal`'s real `CommitWritten` wiring and the `fabric` fix, which the fake-seam unit rows cannot; neither duplicates an existing test.
- Verify coverage: `go build`, `go test ./...`, `go test -tags integration ./...` and `go vet -tags smoke ./...` run the tests of every targeted package.
- Granularity: card 7 is one surface (`lyx config`) with its PATTERN entry and docs, and card 6's two primitives serve one editor path, so neither needs splitting.
