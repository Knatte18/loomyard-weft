# Plan round 1 — fixer report

Every non-decision finding is fixed; F5, F6 and F17 are `decision`-class and went to the parent.
`lym loom validate-plan` passes after every change.

F7's new card was inserted at position 7 with `lym loom insert-card`, so every card from the old card 7 on moved up by one: old 7–19 are now 8–20.
Card references below use the new numbers; every bare "card N" mention in the plan was renumbered by hand.

## Changes

- **F1** (as proposed): card 19 (heddle-cli) creates `RunCLI` and `RunCLIIn`, with an Intent sentence saying they run the subtree from the process cwd and from a given cwd.
- **F2** (as proposed): card 2 lists `internal/runproc#allowedImports` under Edit; its Intent adds `internal/lymcwd` to the allowlist and names it in the comment and file header; the ImpactSummary says so.
- **F3** (as proposed): `TestMain` Create lines and Intent paragraphs in card 11 (entrymerge: `tmuxkit.Main` only, since the package spawns no git), card 14 (heddleengine) and card 19 (heddlecli), both of which call `gitkit.HermeticGitEnv()` and then `tmuxkit.Main`.
- **F4** (as proposed): card 8's guard bullet carries `//lym:guard` directly above the func line.
- **F7** (as proposed): new card 7, `module-bolt-dir-uses`, lists the enclosing members of every production `fabricengine.BoardDir(` call outside fabric whose value reaches the Bolt checkout and not board data, which goes beyond the sites the review named (all found with grep, every glyph resolved with `lym quarry resolve`).
  It also lists the `doc.go` files the validator's doc-go rule asks for, under Edit or Uses.
  Its Intent states the rule for sorting Bolt-checkout uses from board-data uses, where variables split, and what stays.
- **F8** (fixed as proposed, first option): card 10 lists `Fabric.FastForwardPair` under Edit, has a Where-it-fires bullet (log only) and adds a `TestWorktreeSeam_Events` row.
- **F9** (as proposed): card 9's Reconcile row also removes a junction and asserts the repair and the re-seed; the wireJunctionSet bullet says reconcile seeds only a pair it re-wires.
- **F10** (as proposed): card 19 has an Intent paragraph on what `set` and `remove` let through and what bounds them, checked against D4's exceptions.
- **F11** (as proposed, first option): card 13's `TestBoltReplay_NamesNoBoardFile` also scans the bodies of the named readers in `pull.go`, `pushclassified.go`, `originreconcilemoves.go`, `originreconcile.go` and `bolt.go`, and fails when one of them is not found.
- **F12** (as proposed): card 9 loads the config first in `wireJunctionSet`, before `seedLymJunction`.
- **F13** (as proposed): card 19 adds a `TestHeddleCLI` row running `list` from the Bolt checkout and from the records sibling.
- **F14** (as proposed): card 13 moves `restoreReplayTip` from Uses to Edit; its message says "after a failed Bolt replay".
- **F15** (as proposed): card 1 lists the three pattern files under Edit, with an Intent paragraph covering the four lines; a grep of `pattern/` and `PATTERN.md` finds no other hit.
- **F16** (as proposed): card 9's Intent has the `config.go` file header name the exclude list, riding on the card's `config.go` members.
- **F18** (as proposed): `ExcludeWayForward` and its doc.go note moved from card 15 to card 9; card 15 lists it under Uses and no longer extends `fabricengine/doc.go`.
- **F19** (as proposed): card 11's `TestTable` keeps only the uncovered rows and names `TestFabricReplay` as the cover for the rest.
- **F20** (as proposed): card 6 names `yamlengine#TestSetValues` and `vscode#TestBuildWorkspace` as the `lymcwd.BoltDirName` takers. `go list -deps` confirms no other listed in-package test is in fabricengine's dependencies.

## Self-check

- fix-consistency, LOW: overview index line and card 10's ImpactSummary omit the FastForwardPair firing. Fixed.
- fix-consistency / fix-affected-set / fix-claims (merged), LOW: card 7's `Hub.BoardDir` sentence wrongly credits card 6, and hubforge's `SeedFabricConfig` and `CopyHub` reach the Bolt checkout. Fixed: card 7 edits both and `hubforge/doc.go` (card 5's Use of that doc.go dropped, as the validator's uses-later-target rule requires), and says why the accessor stays.
- fix-claims, LOW: `dobbyengine.Geometry.BoardDir` fill is unaccounted for. Fixed: card 7 names it as staying.
- fix-consistency, LOW: card 2's ImpactSummary omits the allowlist change. Fixed.
- fix-claims, LOW: `HubDeps` has no variable feeding both kinds. Fixed: dropped from card 7's split list.
- fix-affected-set, LOW: `Bolt.logReplay` was missing from the widened guard. Fixed: added to card 13's scanned functions.
- fix-proposal, LOW: card 10's FastForwardPair bullet does not say why D6's landing exception does not apply. Fixed: the bullet states that the exception covers commits brought into a beam, and this call moves the parent pair to origin.
- fix-proposal, LOW: heddlecli's seam functions are not pinned in `cmd/lym/seamsignature_test.go`. Deferred, see below.
- fix-proposal, LOW: F19's cut leaves `Table.Change` untested between cards 11 and 12. Disputed, see below.

## Disputed

- fix-proposal's F19 coverage-gap finding, ground `every fix worse`.
  Restoring the four rows re-creates the duplication with `boardengine`'s `TestFabricReplay` (`internal/boardengine/replay_test.go:168,185,200,329`) that F19 removed.
  The testing rule calls a redundant test a defect.
  The gap spans one card: card 12 runs that test over the merge, and the plan's cards land together.

## Deferred

- fix-proposal's seam-pin finding.
  `cmd/lym/seamsignature_test.go` holds only blank-identifier vars, and `lym quarry toc` reports no symbols for it.
  `validate-plan`'s `edit-test-file` rule refuses the file as an Edit target and asks for member glyphs, which this file does not have.
  So no card can list the change.
  It needs the parent, or a later round, to rule how such a file is targeted.
  Nothing fails meanwhile: no guard enforces the file's completeness.
