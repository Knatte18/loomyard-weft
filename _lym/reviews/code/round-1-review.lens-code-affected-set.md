```yaml
lens: code-affected-set
outcome: reported
```

I covered the `code-affected-set` lens: everything the diff's changes reach outside the lines it touches.

**Per change, the set found and whether it is covered**

1. **`ImportResult` gains `EmptyBody` and `LocalPaths`** (`internal/boardengine/intake.go:56-62`).
   - The only consumer outside its package is `boardcli.intakeImport` (`internal/boardcli/intake.go:318-323`). It reads both fields. Covered.
   - The re-read in the critical section, `board.go:155-157`, copies only `r.Entry` and keeps the new fields. Covered.
   - The type is documented in the package doc at `internal/boardengine/doc.go:452-455`. Covered.
2. **Import entry body gains import notes** (`issueContent`, `intake.go:200-216`).
   - Readers of the body format: `TestImportIssueNewEntry`, plus `TestImportIssueFold` and `TestIntakeImport_FoldIntoEntry`, whose default body is non-blank and names no `.lym` path. All are unchanged and still hold.
   - Nothing in production parses the "Imported from" line or the `## From issue #` heading: a search found `intake.go:201` only.
   - The board skill text, `plugins/ly/skills/board/SKILL.md:238-239`, describes the note. Covered.
3. **`intake import` output gains `empty_body` and `local_paths`.**
   - Docs: the `boardcli/doc.go:184-189` output shape, the `boardcli/intake.go:94-95` Long text and `SKILL.md:238`. All covered.
   - Consumer stencil: `contracts/orchrole/role-intake.md` binds the intake fork's outcomes and its one-line-per-issue return. It is not updated (finding F1).
4. **`lym selfreport create --attach`** (`internal/selfreportcli/cli.go`).
   - Docs: the package doc in `cli.go:9-10`, the Long example, the flag help, and `docs/overview.md:621-622`. All covered.
   - Callers: `friction-template-reflection.md:59-62` adds the `--attach` guidance. `internal/friction/doc.go:49` and `contracts/orchrole/orch-role.md:347` name only the verb, so they stay correct.
   - Guards: `internal/shellguard` has no selfreport row, so nothing refuses the new flag.
5. **Friction reflection stencil wording** (`friction-template-reflection.md:57-62`).
   - It is additive. It drops, weakens or contradicts no existing rule; its new last line defers to the content rule.
   - Stencil wording is checked against the amended entry's third bullet, which it carries. Covered.
6. **Repository rules.**
   - `PATTERN-lym-files` (single declarer): production code names `.lym` only through `lymdirs.DotLymDirName` (`intake.go:163,209`, `boardcli/intake.go:94`). The other mentions are prose and test data. Covered.
   - `PATTERN-code-conventions`: the new flag name is a constant (`attachFlag`), and so is the limit (`maxIssueBodyRunes`). Covered.
   - `PATTERN-leaf-packages`: the new `boardengine` → `lymdirs` import is not into a listed leaf. No issue.
7. **Commit messages and departures.**
   - The four messages over `9c5bd131..HEAD` match their diffs.
   - No commit carries a `Pattern-Departure:` or `Loosened-Check:` trailer, and none is needed. Covered.
8. **Declared PATTERN changes:** the directive declares none, so there is nothing to check.

**Findings**

- **F1 — LOW, consistency, `contracts/orchrole/role-intake.md:46-47`.**
  The intake fork returns one line per issue: number, slug or fold or close, labels. The diff adds `empty_body` and `local_paths` to `intake import`'s output, but this stencil's return line does not pass them on. So an import flagged for an empty body or lost `.lym/` evidence reaches the orch as an ordinary new note.
  The flag survives only as the note in the entry body, and no session that acts on intake ever hears of it.
  Evidence: the diff adds the keys at `internal/boardcli/intake.go:318-323` and documents them at `SKILL.md:238-239`. `role-intake.md:46-47` is unchanged.
  Fix: add the gap to the fork's per-issue line, e.g. "then `empty body` or the local paths when import flags them". Alternatively, record that the entry note alone is the flag.

No other member of the affected set is left wrong.

**Tool calls**

1. `grep` for readers of the import body and output keys, `body-file` docs, and the `PATTERN-leaf-packages` text. The manifest had no search for consumers outside the touched files.
2. `docs/getting-started.md` and `contracts/orchrole/role-intake.md`, plus a grep for `Import note`, `empty_body` and `local_paths` elsewhere. These look for consumers of the new output that the manifest did not name.
3. `grep` of `docs/overview.md` for intake, of plugins and specs for selfreport, and of `selfreportengine` for limits. These check docs describing the changed verbs.
4. `docs/overview.md:466-480` and a trailer grep over the commit range. They check the intake doc paragraph and the commit trailers.
5. `grep` of shellguard, heddle, friction and stencils for selfreport. It checks guards and stencils that the new flag could affect.
