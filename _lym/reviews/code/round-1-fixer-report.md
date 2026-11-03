# Fixer-report, code review round 1

## Changes

- **F1 (BLOCKING), fixed as proposed.**
  `configengine.Dirs` gains `Records`, the told path of the records checkout (the weft checkout) paired with the anchor's worktree.
  `fabricengine.ConfigDirs` fills it from `RecordsWorktree` when that directory exists, and `TestConfigDirs` pins both cases.
  `writableRoots` and its link reading are gone from `codexengine/home.go`; `Prepare` passes `Dirs.Records` as the one writable root when it is non-empty.
  The root is the records checkout, which covers the `<records>/<AnchorRel>` link targets of a non-`.` anchor.
  Updated the `Prepare` and `Codex.dirs` comments, `doc.go`, `_lym/landing/summary.md`'s Design decisions entry and `docs/overview.md`'s shuttle entry.
  The fixture no longer symlinks `.lym` and `_lym`; `TestPrepare_Launch` supplies the path through `engine.dirs.Records`.
  The fix named `shuttleengines.New` as the place to fill the path, but `New` receives only `Dirs`, and every hub caller builds its `Dirs` through `fabricengine.ConfigDirs`, so the path is filled there. This is a different place than proposed and is not a disputed finding.
- **F2 (LOW), fixed as proposed.**
  `ConfigTemplate`'s comment no longer enumerates the aliases.
- **F3 (LOW), fixed as proposed.**
  Rows in existing tests: `TestPrepare_Launch` (`RecordsTold`, `RecordsNotTold`: no writable-roots setting), `TestPrepare_ReadiesHome` (`PlainClone`: one trust root) and `TestPrepare_RefusesBeforeWriting` (`CommittedRulesUnderSubfolderAnchor`).
  The real `.lym`/`_lym` directory case reduces to the not-told row, since the engine reads no link now.
  The fixture gained `addClone`.

Gate repairs after the first pass:

- Two comments wrapped at a fixed column, `ConfigTemplate`'s in `internal/modelspec/template.go` and `addWorktree`'s in `fixture_test.go`, are broken at sentence boundaries.
- `TestGofmt_ModuleFilesAreFormatted` flagged `internal/fabricengine/junctionnames_test.go`, which `lym loom fmt` reformatted.
- `TestEnforcement_FabricVocabulary` flagged bare weft tokens outside the owner set in `docs/overview.md`, `configengine/layers.go`, `codexengine/codexengine.go`, `doc.go`, `fixture_test.go` and `prepare_test.go`.
  The field is renamed `Dirs.Records`, and the comments, test names, test field and fixture names use "records" in place of the bare token; `fabricengine`, an owner, keeps its own wording.
  `lym gate test` over `lymcwd`, `codexengine`, `fabricengine`, `configengine` and `cmd/lym` then passed at tier 0 (1770 passed, 0 failed).

`_lym/landing/summary.md` is edited in the working tree and not committed, per the rule against git commands on `_lym` paths.

## Self-check

Lens names below are as the forks reported them; the fixes for their findings predate the rename.

- fix-code-correctness, MEDIUM: the weft path is filled only by `ConfigDirs`, so other launch paths get no writable root. Disputed, premise false (below).
- fix-affected-set, MEDIUM: `docs/overview.md` shuttle entry still described the link derivation. Fixed as proposed.
- fix-consistency: no findings.
- fix-proposal: no findings.

## Disputed

- **fix-code-correctness MEDIUM**, ground `premise false`.
  The hand-built `Dirs` it names are not hub paths: `loomcli/decision.go:1144`, `loomcli/looparm.go:105`, `pairteardown/cancel.go:237` and `battencli/cancelcmd.go:431` all build `Dirs` with `fabricengine.ConfigDirs`.
  Only `burlercli/wiring.go:211` hand-builds `Dirs{Anchor: res.StateDir}`, a standalone state directory outside a hub with no records sibling, where no writable root is the intended behaviour.

## Deferred

None.
