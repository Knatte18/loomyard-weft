# Plan-Review round 2 — fixer report

Every finding's premise held against the code and the decision record; all five are fixed.

## Changes

- **F1** (`01-shedrun-tree-records.md`, Intent): fixed as proposed.
  The card now names `SeedRel`, `StatusRel` and `DriveReportsRel` beside `RunsRootRel` as constructors that already break the `doc.go` and `paths.go` claim (`internal/shedrun/paths.go`: each is a relative `filepath.Join` that never touches `AnchorPath()`).
  The narrowed claim is worded by return shape: absolute-path constructors join onto a told `Location`'s `AnchorPath()`, the `*Rel` constructors and `TreeRunFileRels` return relative paths, and no constructor resolves a cwd or spawns git.
  The first sentence of `paths.go`'s header, which lists the kinds of path, gains the tree-relative seed and status paths.
- **F2** (`04-shed-span-producers.md`, Intent and Tests): fixed as proposed.
  The Intent parses the seed's `started_at` when present, every entry's `at`, and `last_at` only when `Repeats` > 0, since `shedengine.HistoryEntry.LastAt` is `omitempty` and empty on an unfolded entry (`internal/shedengine/status.go`).
  The first test row adds an unfolded entry with no `last_at` that reads without error.
- **F3** (`04-shed-span-producers.md`, Intent): fixed differently.
  The run-id error is wrapped with the slug, the tag's `Name` and both files, `seed.json` and `status.json`, in the decode errors' shape, and stays a fail-loud guard with no test row.
  The proposed fix named "the `shed` run directory" in the message; it is not made because the tool would then spell the `shed` segment, which `PATTERN-shed-run-directory` and the decision record's constraint reserve to `internal/shedrun`.
  A validation failure precedes both file reads, so naming both files is what Decision 9's "the file" can mean there.
- **F4** (`05-run-reads-shed-status.md`, Intent): fixed with the second option the review offered.
  The Intent states that `resolveHub` takes `filepath.Dir(wd)` as the worktree's parent, assuming the tool runs from the worktree root as the `-hub` default already does (`tools/tokencount/main.go`) and as both launchers ensure (`tokencount.sh` and `tokencount.cmd` change to the repository root).
  It names the subdirectory consequence and the weft flag as the way through, and the refusal now names the current directory's parent as the hub it assumes.
  Comparing against `lyxcwd.Resolve(wd)`'s worktree parent was not chosen: the `-hub` default stays `filepath.Dir(wd)`, so from a subdirectory the default hub itself would be refused.
- **F5** (`03-archive-tag-dates.md`, Tests): fixed as proposed.
  `TestArchivedSlugs` gains a skipped row: a well-shaped ref with an empty object-name field and a valid date.

## Disputed

None.

## Deferred

None.
