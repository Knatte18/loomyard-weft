# Card 7 — inbox-github-calls

**Edit:**
- `internal/selfreportengine/selfreport.go`

**Create:**
- `internal/selfreportengine/inbox.go`
- `internal/selfreportengine/inbox_test.go`

**Intent:**
D11: `selfreportengine` stays the sole owner of the inbox repo constant `targetRepo` and gains the three GitHub calls `lyx board intake` needs, through the same `NewGitHubClient` seam, so authentication stays `githubclient.New` (PATTERN-github-auth).

`inbox.go`:
- `Issue`, the JSON-tagged view intake prints and decides on: number, title, body, label names, URL, creation time, state, state reason, and whether it is a pull request.
- `ListOpenIssues() ([]Issue, error)`: every open issue of `targetRepo`, following pagination until the last page, with pull requests excluded.
- `GetIssue(number int) (Issue, error)`: one issue or pull request, open or closed.
- `CommentAndClose(number int, comment string, completed bool) error`: posts `comment`, then closes the issue with state reason `completed` when `completed` is set and `not_planned` otherwise.
  When the comment posts and the close fails, the error says the comment was posted, so a rerun of `intake close` is understood to post a second one.
- Each call is bounded by a timeout as `CreateIssue` is, logs a failure through `internal/logger` as `CreateIssue` does, and classifies errors the same way; extract `CreateIssue`'s classification into one helper in `selfreport.go` that all four calls share.

`selfreport.go`: the package doc and `targetRepo`'s comment say the constant now serves both directions, filing and intake.

Tests (`inbox_test.go`, untagged against an `httptest` server through `NewGitHubClient`, as `selfreport_test.go` does, per PATTERN-test-tier-purity): pagination across two pages via the `Link` header, a pull request excluded from the list, the exact owner/repo path, `GetIssue` reporting a pull request and a closed issue's state, `CommentAndClose` sending the comment and then the close with each state reason, a failed comment that skips the close, a failed close after a posted comment, and an unresolvable token.

**ImpactSummary:** Adds three calls to selfreportengine and extracts one shared error helper; `CreateIssue`'s behaviour and messages are unchanged.
