# Card 1 — parse-owner-repo-shapes

**Edit:**
- `internal/githubclient#ParseOwnerRepo`
- `internal/githubclient#ownerRepoPath`
- `internal/githubclient#TestParseOwnerRepo`

**Uses:**
- `internal/githubclient/doc.go`

**Intent:**
D4: a pull request needs an origin whose owner and repository `ParseOwnerRepo` reads, so the parser learns the two remaining github.com shapes.
`ownerRepoPath` also strips the `ssh://git@github.com/` prefix and an `https://<user>@github.com/` prefix, where `<user>` is one non-empty run of characters other than `/`, `@` and `:`, so `https://Knatte18@github.com/Knatte18/loomyard.git` reads `Knatte18`, `loomyard`.
Both new shapes keep the optional `.git` suffix and the single optional trailing slash, as the two existing shapes do.
Any other URL holding `://` or `@` keeps the existing "not a github.com remote URL" error, so `ssh://git@gitlab.com/o/r.git` and `https://u@gitlab.com/o/r` are still refused, and `https://@github.com/o/r` (empty user) is refused as not a github.com remote.
`ParseOwnerRepo`'s doc comment names the four accepted shapes.
`TestParseOwnerRepo` gains rows: `ssh://git@github.com/Knatte18/loomyard.git`, `ssh://git@github.com/Knatte18/loomyard`, `https://Knatte18@github.com/Knatte18/loomyard.git`, `https://Knatte18@github.com/Knatte18/loomyard` (each reads `Knatte18`, `loomyard`), and the refused `ssh://git@gitlab.com/Knatte18/loomyard.git`, `https://user@gitlab.com/Knatte18/loomyard` and `ssh://git@github.com/loomyard` (no owner/repo pair).
The package doc names no URL shape, so it stays as it is.

**ImpactSummary:** One pure parser and its table test; its callers (`fabriccli`, `hubissues`) only accept more URLs, and every URL accepted today reads the same.

**Commit:** 1: githubclient: ParseOwnerRepo reads the ssh:// and user-info https:// shapes
