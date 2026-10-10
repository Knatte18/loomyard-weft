# Card 11 — intake-comments-body

**Create:**
- `plan:internal/boardengine#IssueComment` -> `type IssueComment struct`

**Edit:**
- `internal/boardengine#InboxIssue`
- `internal/boardengine#issueContent` -> `func issueContent(issue InboxIssue, commentHeading string) string`
- `internal/boardengine#Store.importNote`
- `internal/boardengine#Store.foldIssue`
- `internal/boardengine/intake_test.go`

**Intent:** Carry an issue's comments into the board body (D11), on the engine side, with no GitHub knowledge.
`IssueComment` holds `Body string` and `CreatedAt time.Time`; `InboxIssue` gains `Comments []IssueComment`, already filtered and ordered oldest first by the caller.
`issueContent` takes the heading prefix for comment sections and appends, after the issue body, one section per comment: the heading `<prefix> Issue comment, <YYYY-MM-DD>` with the UTC date of `CreatedAt`, a blank line, then the body verbatim, author not recorded.
`importNote` passes `## `, so a new note's comments sit at the second level; `foldIssue` passes `### `, so under its `## From issue #N` heading each comment sits one level deeper.
`TestImportIssueNewEntry` and `TestImportIssueFold` gain comment rows pinning the exact rendered body for two comments in order at each level, and the empty-comments case keeps today's body byte for byte.

**ImpactSummary:** One helper re-signed with its two callers in the same file and one struct extended; a board body with no comments is unchanged.
