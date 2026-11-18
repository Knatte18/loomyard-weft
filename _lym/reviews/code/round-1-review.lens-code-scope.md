```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. Trace every change in the diff to a goal point of the board entry, and every goal point to the changes that build it.

**Decisions.** The answer key is the decision record alone. Its `## Goal` holds three goal points, and the record has no `Added after Discussion` entries.

- **G1:** on board import, copy the evidence the issue names into the entry, or into a durable file the entry references.
- **G2:** flag an import whose issue body is empty.
- **G3:** consider a hint at issue-creation time (`lym selfreport create`) to include captured evidence inline.

**Per change**

- **C1 – local-path detection and import note.** Covers `localPathPattern`, `localPathsNamed`, the second import note in `issueContent`, and `ImportResult.LocalPaths` in `internal/boardengine/intake.go`, plus its `doc.go` paragraph and the `TestImportIssueNewEntry` paths row. Traces to G1, but only in part: it flags the named evidence instead of copying it (see F1).
- **C2 – empty-body flag.** Covers `issueBodyEmpty`, the empty-body note in `issueContent`, `ImportResult.EmptyBody`, the `doc.go` paragraph and the `TestImportIssueNewEntry` blank-body row. Traces to G2.
- **C3 – `intake import` output keys.** Covers `empty_body` and `local_paths` in `internal/boardcli/intake.go`, the `Long` help text, `boardcli/doc.go`, and `TestIntakeImport_FlagsEvidenceGaps` with the `fakeIssue.body` fixture field. Forced by C1 and C2 (G1 and G2).
- **C4 – `plugins/ly/skills/board/SKILL.md` lines.** Forced by C3, because the skill documents `import`'s output.
- **C5 – `lym selfreport create --attach`.** Covers `appendAttachments`, `attachmentSection`, the flag, the help example, the package doc, the new `TestRunCreate_Success` and `TestCommand_RefusedBeforeTransport` rows, and `docs/overview.md`. Traces to G3; the help example and the overview line are the hint, and `--attach` is the means it points to.
- **C6 – friction reflection stencil.** Covers the "carry run-directory evidence inline" paragraph in `contracts/stencils/friction/friction-template-reflection.md:59-62`, which traces to G3. The "Never file an issue with an empty body" line (`:57`) traces to G2 and G3 together: G2 addresses the empty-body loss, and an empty body cannot carry evidence inline. I judge it in scope and raise no finding.

No hunk traces to nothing, and no commit in the range carries a `Pattern-Departure:` trailer.

**Per decision**

- **G1:** C1 and C3 flag the evidence. C5 makes inline evidence possible at filing time, and the import then carries the issue body verbatim. No change copies a named file into the entry or into a durable file.
- **G2:** C2, C3 and C4, plus C6's prevention line.
- **G3:** C5 and C6.

## Findings

**F1 – MEDIUM, scope**
- **Where:** `internal/boardengine/intake.go:152-184` and `:200-215` (`localPathsNamed`, `issueContent`).
- **Summary:** G1 asks the import to copy the evidence the issue names into the entry, or into a durable file the entry references. The diff copies nothing: it adds an import note listing the `.lym/` paths and returns them in `local_paths`.
- **Why it is a finding:** the substitution is the writer's own call in `_lym/landing/summary.md`, "Design decisions", first bullet ("The import flags local paths instead of copying the files"). No `Added after Discussion` entry or parent decision records it; the record's `## Decisions` section is empty.
- **Evidence:** the board entry's first bullet under "What to do" in `_lym/discussion/decision-record.md`. The diff's `issueContent` writes only "Import note: the issue names files under `.lym/` … only what the issue quotes of them reached the board" (`internal/boardengine/intake.go:209-211`).
- **Fix:**
  - Either make the import copy each named path it can resolve, such as a file still present under the importing hub's worktrees, into a durable file the entry references, and flag only the paths it cannot resolve.
  - Or leave the code in place and defer to the run's parent, who records the "flag instead of copy, copy at filing via `--attach`" substitution with `lym loom decision add`.
  - The summary's reasoning (the path is relative to an unknown worktree on an unknown machine, and the worktree is usually gone) makes the second route the likely one.
- **Severity:** I rate it MEDIUM because C5 moves the durable copy to filing time, which the entry's own third bullet invites. Strictly, the entry's literal ask at import is unbuilt, so the handler may raise it to BLOCKING.

Tool calls: none. The inherited context already held the decision record, the summary, the commit log and every diff piece.
