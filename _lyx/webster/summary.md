# pattern-init: invariants move to PATTERN.md

All 23 batches reported `status: done`, in the listed order, with no recovery needed.

## Fabric residue and PATTERN delivery (01–04)

- Batch 01 removed fabric's PATTERN-residue report and its `pattern_residue` pull-envelope key.
  It also cleaned three comment mentions in `unwire.go`, `junction.go` and `unwire_test.go`, reported as deviations.
- Batch 02 made `pattern.Directive` and `pattern.File` read `PATTERN.md` at the warp worktree root and inline it into a `{{.pattern_overview}}` marker.
  An absent or whitespace-only file is inactive.
  Burler and webster gained a told `RepoRoot` field.
- Batch 03 added `RoleDesigner` on Discussion-Write.
- Batch 04 added `RoleJudge` on the Bouncer judge and rewrote the role doc.
  It reported two test files in `internal/shedadapters` as deviations, switched to `stencil.FillOptional`.

## CONSTRAINTS.md retired (05, 07–10, 13)

- Batch 05 wrote `PATTERN.md` and the `pattern/PATTERN-*.md` background files from the entry-map.
  Many entry lines exceed 200 characters but stay under the 320 cap.
- Batches 07–10 repointed stencil, `cmd/lyx`, tools and internal citations to `PATTERN-*` entries.
- Batch 13 repointed docs, specs, crucible and the skill, then deleted `CONSTRAINTS.md`.

## manifest/ retired (11–12)

- Batch 11 moved unbuilt designs onto the board as notes and built ones into package docs, including a new `internal/orchcli/doc.go`.
- Batch 12 deleted `manifest/`, fixed inbound links and rewrote the docs lifecycle text.
  `docslink_test.go` now scans `docs/` only.

## Vocabulary sweep and scan (14–18)

- Batches 14–17 reworded weft/warp in tests and docs.
  Batch 14 reported `pattern/PATTERN-batten-bookend.md` as a deviation for a renamed test.
- Batch 18 widened the fabric-vocabulary scan with a closed allowlist and the 13 `vocab-exempt` tokens.
  It added further exempt spellings reported by cards 14–17, each with a reason naming its declaration.
  It also renamed `warpDirName` to `repoDirName` in `tools/sandbox`.
  Reported deviations: `boardtest/sync_test.go`, `pattern/pattern_test.go`, `shedcli/seed_test.go`, the `ly-drive` `SKILL.md` and the `tools/sandbox` files.

## Remaining cards (19–23, 06)

- Batch 19 made the short model-spec form canonical and linked the templates to the spec.
- Batch 20 gave each spawn site a role constant and added the literal-role scan in `cmd/lyx/spawnrole_test.go`.
- Batch 06 added the pure PATTERN format checker and the real-file test.
- Batch 21 told the prime workspace path through `WorktreePath`, kept the portal anchor, and listed existing folders only.
- Batch 22 set the ide spawn log level by target and added the relative-setting warning.
  The fork also made `writePrimeWorkspace` write twice, because card 21's existing-folders filter dropped `_portals` on a first prime spawn.
- Batch 23 dropped `Seed.Parent` and its fallback while old seeds still decode.
  Reported deviations: `battencli/lifecycle_integration_test.go`, `loomcli/parentreview_wiring_test.go`, `loomcli/wiring.go`, `hubgeom/doc.go` and `shedrun/doc.go`.

The plan-level verify gate runs next.
