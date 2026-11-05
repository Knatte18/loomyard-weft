```yaml
lens: fix-code-correctness
outcome: reported
```

Lens fix-code-correctness: I ran each changed function and each added test, and found no findings.

- **`scratchLinkedInSibling` (F1):**
  - An entry that exists gives a nil error, so `!errors.Is(err, fs.ErrNotExist)` is true and the function answers true.
  - A missing path answers false and the loop continues.
  - Any other error, such as ENOTDIR or a permission error, answers true, which is the conservative result the doc comment states.
  - The `fslink` import is still used by `wireAnchorScratchLink`, `restoreScratchTarget` and `removeOwnedScratchLink`, and the gate built and passed.
  - Both callers still hold. `retireRootScratchLink` passes `"."`, and `unwireAtSource` passes the source anchor.
- **F1 row `RealRootDirectoryOfASiblingKeepsTheRootExcludeLine`:** the pair's legacy root link is retired and the prime holds a real root `.scratch` directory. With the old `IsLink` census the prime counts as no link, so `/.scratch` would be dropped and the final exclude assertion would fail. The row catches the defect.
- **F5 row `LegacyRootLinkAtASubpathAnchorIsUnlinkedWithTheAnchorLink`:** `wireLegacyRootScratchLink` removes the anchor link, and the row recreates it, so both links exist before `Remove`. Without the `!anchorIsRoot` branch there is no root `link_removed`, so the `len(rootUnlinked) != 1` check fails.
- **F6 rows (directory, file, foreign link):** the anchor link is removed first, so the assertion that wiring still creates it can fail. `replaceExcludeLine(..., "/.scratch", "/.scratch")` seeds the line, so a line wrongly dropped is caught. The second `Reconcile` reports the left entry again, so the `Detail` check holds.
- **F7 anchor loops:**
  - `trackScratchFile` now computes its path with `filepath.Rel`, so it is correct at `backend`.
  - `scratchExcludeLine` fixes the `/.scratch` assumption in `writeLegacyScratchExclude` and in the adopt assertions.
  - The gate ran the `_Anchor_backend` rows and they passed.
- **F4 subtest:** the prime is re-anchored to `sub`, then the mover moves from `.` to `other`. The census finds no sibling at `.`, so `/.scratch` drops. Deleting the `unseedScratchExclude` call in `unwireAtSource` would leave `/.scratch` in place and fail the test. The extra `/.scratch` and `/sub/.scratch` entries in the first subtest's want list hold because the stayer and the prime still link `.`.
- **F3 test:** the child sits under `&lt;prime&gt;/backend/.scratch`. If the segment read the worktree root, it would not find the child (or would fail on an absent directory), and the child would survive, so the `exists(scratchChild)` assertion fails. The cwd is `AnchorPath()`, which the strict cwd gate requires.

Two points outside my lens, not findings:
- **F8 and `DoneChecks`:** the F8 spec edit now says "worktree anchor's `.scratch/`", but `scratchDirPrefix` in `internal/planglyph/donecheck.go` still skips only the root-relative `.scratch/` prefix.
- **Commit grouping:** the F1 `doc.go` wording went into the F2 commit.

Further tool calls: none, because my context already held everything this lens needed.
