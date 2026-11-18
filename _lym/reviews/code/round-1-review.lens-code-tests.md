```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. I mapped each behaviour the diff adds or changes to the test row that would fail if it broke.

**Behaviour → pinning test**

- Blank or whitespace issue body gets an import note naming the title (new note) → `TestImportIssueNewEntry` "blank body flagged with the title" (`internal/boardengine/intake_test.go:116-122`).
- `ImportResult.EmptyBody` is set for a blank body (new note) → the same row, at `intake_test.go:156`.
- `.lym/` paths in the body and comments get an import note listing them, and `LocalPaths` is set. The cases are first-mention order, deduplication, trailing punctuation trimmed, a `.lym/` mid-path component, and `_lym/` not matched. All pinned by `TestImportIssueNewEntry` "local paths in body and comments flagged" (`intake_test.go:123-136`, `:159`).
- Blank body and local-path notes on the **fold** path (`foldIssue` → `issueContent(issue, "### ")`) → **none**. `TestImportIssueFold` (`intake_test.go:166-212`) only folds `testIssue`/`testIssueWithComments`, which have a non-blank body and no `.lym/` path.
- `EmptyBody`/`LocalPaths` set on the **fold** result (assigned in `importIssue` after the branch, `intake.go:148-149`) → **none**. No fold test asserts either field, and no `intake import` fold row in boardcli checks `empty_body` or `local_paths` (`internal/boardcli/intake_test.go:479`).
- The already-recorded no-op returns no flags → none. No designed behaviour depends on it, so I filed no finding.
- `intake import` prints `"empty_body":true` and `"local_paths":[…]`, and leaves each key out when its gap is absent → `TestIntakeImport_FlagsEvidenceGaps` (`internal/boardcli/intake_test.go:432-467`). Each row checks one key present and the other absent.
- `--attach` appends sections after the body in order, under the base name (`./report.md` → `report.md`), with the body's and the content's trailing newlines trimmed → `TestRunCreate_Success` "attachments follow the body" (`internal/selfreportcli/cli_test.go:201-203`).
- A fence longer than any backtick run in the file → the same row, using `fencedFileName`.
- An attachment with no body becomes the whole body (nil body) → "attachment alone is the body" (`cli_test.go:204-205`).
- `-b ""` with `--attach` gives no empty leading section (the `*body != ""` branch at `cli.go:201`) → **none**.
- A missing attachment is refused before the transport → `TestCommand_RefusedBeforeTransport` "missing attachment" (`cli_test.go:336`).
- A body over the limit is refused before the transport → "attachment over the body limit" (`cli_test.go:337`). It pins only a file of `maxIssueBodyRunes+1` characters, which is over the limit on its own. Three things stay unpinned:
  - that the check measures the whole joined body (body plus headings), not each file alone;
  - the `&gt;` versus `&gt;=` boundary;
  - counting runes rather than bytes, since all the content is ASCII.

**Findings**

1. **LOW · design · `internal/boardengine/intake_test.go` `TestImportIssueFold`.** The fold path's new behaviour has no test: the import notes inside the `### ` section, and `EmptyBody`/`LocalPaths` on a fold's `ImportResult`. Moving the flag assignment into `importNote`, or calling a variant of `issueContent` without the notes for folds, would pass every test. Evidence: `intake.go:138,148-149,292`; `intake_test.go:166-212` only folds `testIssue` or `testIssueWithComments` bodies. The summary claims the fold keeps the title (`_lym/landing/summary.md:21`).
2. **LOW · design · `internal/selfreportcli/cli_test.go:337` "attachment over the body limit".** The size check is pinned only by one file that is over the limit by itself. Three implementations would still pass:
   - one that checks each file's length instead of the joined body;
   - one that uses `len` (bytes) instead of runes;
   - one that refuses at exactly the limit.

   A row whose file fits the limit but whose total with the body and heading exceeds it would pin that the whole body is measured, which `cli.go:211-212` claims. Evidence: `cli_test.go:156` (`strings.Repeat("x", maxIssueBodyRunes+1)`), `cli.go:211-215`.
3. **NIT · design · `internal/selfreportcli/cli_test.go` `TestRunCreate_Success`.** No row covers `-b ""` with `--attach`, so the `*body != ""` guard at `cli.go:201` is unpinned. Dropping it would put two leading newlines before the first `## Attached` heading, and no test would fail.

Tool calls:
- One grep of `internal/boardengine/intake_test.go` and `internal/boardcli/intake_test.go` for the fold tests and flag assertions. My context held the diff's hunks but not the existing fold tests.
