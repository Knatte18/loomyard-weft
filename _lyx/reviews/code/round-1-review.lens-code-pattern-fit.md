```yaml
lens: code-pattern-fit
outcome: reported
```

Task: the code-pattern-fit lens, limited to the amendment, which is `docs/overview.md` after the Merge-In.

**Rules checked**
- **CLAUDE.md, docs land in the same commit: held.**
  `docs/overview.md` carries the board entry (line 358) and config entry (lines 372-373) profile text, in the task's commit range.
- **CLAUDE.md, semantic line breaks: held.**
  The task's edited lines are one sentence per line, and the split at the `;` in the config entry is at an independent-clause boundary.
  The long gate entry (line 461) is the parent's own text, not the task's.
- **Merge integrity of `docs/overview.md`: held.**
  The gate entry carries the parent's `--card` text.
  The board and config entries carry the task's text.
  `git diff c9c4e7d..HEAD -- docs/overview.md` shows only the task's two hunks.
- **Agreement with `internal/runprofile/doc.go` and `template.yaml`: held.**
  - The ladder is `speed`, `light`, `solid` and `wide`.
  - `docs` is darn-only.
  - A task naming no profile takes its recipe's default.
  - Seed-Child writes the block for the task's recipe.
- **Agreement with `internal/boardcli/doc.go`: held.**
  `lyx board profiles` printing "each recipe's default profile" matches the `defaults` field.
  The module-table row for `internal/runprofile` (line 313) is still accurate.
- **Parent changes since the settle touching the profile code: none.**
  `git diff --stat 04aafa88..c9c4e7d` over `runprofile`, `boardcli`, `configreg`, `battenshed`, `battencli`, `configcli`, `boardengine`, the board skill and the orch stencil is empty.
  Only `docs/overview.md` overlaps, and its parent hunk is the gate entry alone, which has no profile text.
- **Declared PATTERN changes landed: not applicable.**
  The directive declares none.
- **Commit messages and departures: held.**
  No commit in the range carries a `Pattern-Departure:` trailer, and none is needed.

**Findings:** none.

**Tool calls:** none; the manifest context already held everything this lens needed.
