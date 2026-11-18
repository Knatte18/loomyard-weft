```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix change set (fix-consistency lens) for stale twins and sibling disagreements.

**Findings**

1. LOW, `contracts/stencils/burler/burler-diff-pieces.md:8`, "A package too large for one piece is split at file or hunk boundaries into numbered parts."
   - The code now also cuts at line ends: an oversized hunk, or an over-long line.
   - It also repeats the file header on every piece of a split file after the first (`internal/burlerengine/diffpieces.go`, `boundedUnits` and `packUnits`).
   - `doc.go` states both. The stencil, the text the reviewer reads, states neither.
   - A reviewer can meet a piece that opens with a header and a mid-hunk line, with nothing in the stencil to explain it.

Everything else I checked agrees:
- **Production wiring:** the only production `shedrecipe.Env` literal is `internal/loomcli/wiring.go:897`, and it sets `CodeDiff`. The test-built Envs are covered by the `envkit` default plus `nilLegal`.
- **`stage_diff` rows:** the three recipe rows (darn, sampler, tapestry) use only the `stage_diff` key. Their comments name no seam, so nothing there is stale.
- **Subject-patch and size-cap references:** a grep for subject-patch wording and the old 60,000 cap finds nothing left outside `_lym` and the research docs. The comments on `Profile.Diff` and `BurlerHalfDeps.Diff` stay true.
- **`recordedTaskBase` and `codeSubject`:** the extracted helper preserves the standalone not-found behaviour.

Tool calls (3 of 6):
1. `grep` over Go, markdown and YAML for stale subject-patch and cap wording and `diffPieceMax`: confirms no stale twins. My context did not hold the rest of the repository.
2. `grep` over the three recipe YAMLs and the `Diff`-related comments in burlerengine: to read the surrounding comments, which my context lacked.
3. `grep` for `shedrecipe.Env{` and `CodeSubject:` literals: to find any Env construction that omits `CodeDiff`.
