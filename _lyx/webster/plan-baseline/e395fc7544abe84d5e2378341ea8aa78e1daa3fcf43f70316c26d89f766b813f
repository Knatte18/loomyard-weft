# Card 10 — issue-comments-fetch

**Create:**
- `internal/selfreportengine#IssueComment`
- `internal/selfreportengine#ListIssueComments`

**Edit:**
- `internal/selfreportengine#Issue`
- `internal/selfreportengine#toIssue`
- `internal/selfreportengine/inbox_test.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/selfreportengine#repoClient`
- `internal/selfreportengine#ListOpenIssues`

**Intent:** Give intake the GitHub side of issue comments (D11), through the existing `repoClient` seam so no new package calls GitHub (`PATTERN-github-auth`).
`Issue` gains `Comments int`, tagged `comments`, which `toIssue` reads from the issue's own `comments` field (`GetComments`), so `ListOpenIssues` makes no extra call per issue.
`IssueComment` carries `Body string` and `CreatedAt time.Time`.
`ListIssueComments` lists one issue's comments through `Issues.ListComments` with `Sort: "created"`, `Direction: "asc"` and the list page size, following `NextPage` to the end under the shared call timeout and classifying a failure through `classifyCallError`, so the result is oldest first across pages.
`inbox_test.go` adds one test for `ListIssueComments` against the `httptest` server: two pages followed through the `Link` header, the exact `/repos/Knatte18/loomyard/issues/<n>/comments` path with the sort and direction query, bodies and creation times carried over in order, and a rejected response surfacing its message; the paginating list test gains an assertion that `comments` is read from the list response, and `TestInboxCalls_TokenNotResolvable` gains a row for the new call.
`cmd/lyx/testdata/test-budget.yaml` raises `internal/selfreportengine` by one.

**ImpactSummary:** One new field read by every `Issue` consumer (boardcli's intake view, additive) and one new call; no existing call changes shape.
