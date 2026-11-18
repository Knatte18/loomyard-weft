```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I mapped each change in the diff to a goal point of the board entry and flagged anything unmatched.

**Decisions.** The record is a decision record alone with no `Added after Discussion` entries. `_lym/discussion/decision-record.md:1-27` holds three goal points:
- G1: when the board imports an issue, copy the evidence the issue names into the entry, or into a file the entry references that outlives the source worktree.
- G2: flag an import whose issue body is empty.
- G3: consider a hint at issue-creation time (`lym selfreport create`) to include captured evidence inline.

**Per change**
- C1: the boardengine `.lym/` path detection in `internal/boardengine/intake.go:153-184` and `:205-208`. It covers `localPathPattern`, `localPathsNamed`, the `LocalPaths` result field, the import note listing the paths, the `doc.go` lines and the `TestImportIssueNewEntry` rows.
  - Traces to G1, partly: it flags and lists the paths but copies nothing (see F1).
- C2: the boardengine blank-body handling in `intake.go:161-164`, `:202-204` and `:209-211`. It covers `issueBodyEmpty`, the `EmptyBody` field, the import note naming the title, the `doc.go` lines and the test row.
  - Traces to G2.
- C3: the boardcli output keys in `internal/boardcli/intake.go` and its help text. It covers `empty_body` and `local_paths`, `boardcli/doc.go`, `TestIntakeImport_FlagsEvidenceGaps` with the fake's body field, and the board `SKILL.md` lines.
  - Traces to G2 for `empty_body` and to G1 (partial) for `local_paths`.
  - The doc, skill and test edits are forced by C1 and C2.
- C4: `lym selfreport create --attach` in `internal/selfreportcli/cli.go:101`, `:119-123` and `:191-233`. It covers `appendAttachments`, `attachmentSection`, the GitHub body limit, the help example, the package doc, the tests and the `docs/overview.md` bullet.
  - Traces to G3. G3 asks for a hint; a flag that puts evidence inline, with a hint in the help text, is within its intent.
  - The overview and tests are forced by C4.
- C5: the friction reflection stencil in `contracts/stencils/friction/friction-template-reflection.md:57-62`, which tells the agent to carry run-directory evidence inline via `--attach` or quoting, and never to file an empty body.
  - The inline-evidence lines trace to G3, since the reflection agent is the issue creator.
  - "Never file an issue with an empty body" prevents the G2 gap at its source. That is close to the entry's "Why this is abnormal" and stays in scope.

**Per decision**
- G1: C1 and C3 flag the paths; nothing copies evidence. G1 is only partly built (F1).
- G2: built by C2 and C3, with C5 as source prevention.
- G3: built by C4 and C5.

**Findings**

F1 — MEDIUM, class `scope`, location `internal/boardengine/intake.go:194-216` (`issueContent`) and `:148-150` (`importIssue`).
- Summary: G1 asks the import to copy the evidence the issue names into the entry or a durable file the entry references. The diff only lists the `.lym/` paths in an import note and the `local_paths` output key, and copies nothing.
- The narrowing rests on the writer's own "Design decisions" bullet in `_lym/landing/summary.md:29-31`: the importer cannot tell which worktree a path belongs to, and the durable copy moves to filing time through `--attach`.
- No `Added after Discussion` entry or `Pattern-Departure:` trailer records the narrowing. `git log 9c5bd131..HEAD` shows no trailer, and `decision-record.md` has no added entries.
- The `--attach` route does put the attached content onto the board at import, because the body is copied verbatim. It does nothing for an issue that names a path without attaching the file, which is exactly G1's case.
- Fix: either copy the named files the importer can resolve into the entry or a durable `_lym` file the entry references, or have the run's parent record the narrowing with `lym loom decision add`. The fixer cannot record it.
- Evidence: `_lym/discussion/decision-record.md:22`, `internal/boardengine/intake.go:205-208`, `_lym/landing/summary.md:29-31`.

No other scope findings: every other change traces to G2 or G3, or is forced by a traced change.

Tool calls: none. The diff, record, summary, commit log and code were already in my inherited context.
