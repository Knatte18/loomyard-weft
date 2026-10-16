# Card 3 — github-per-request-deadlines

**Edit:**
- `internal/selfreportengine#callTimeout`
- `internal/selfreportengine#ListOpenIssues`
- `internal/selfreportengine#ListIssueComments`
- `internal/selfreportengine#CommentAndClose`
- `internal/selfreportengine#TestListOpenIssues_PaginatesAndExcludesPullRequests`
- `internal/fabriccli#openPRListTimeout`
- `internal/fabriccli#fetchOpenPRHeads`

**Create:**
- `plan:internal/selfreportengine#listOpenIssues` -> `func listOpenIssues(perRequest time.Duration) ([]Issue, error)`

**Intent:**
A GitHub caller whose one context deadline spans several requests derives a fresh deadline per request instead, so each request's own extension from card 2 covers its own hold (D6).

- `ListOpenIssues` returns `listOpenIssues(callTimeout)`, whose page loop derives `context.WithTimeout(context.Background(), perRequest)` once per page, cancelled before the next page.
- `ListIssueComments` derives `context.WithTimeout(context.Background(), callTimeout)` inside its page loop, once per page, cancelled before the next page.
- `CommentAndClose` derives one deadline for the comment and a second for the close.
- `fetchOpenPRHeads` derives `context.WithTimeout(ctx, openPRListTimeout)` per page;
  `openPRListTimeout`'s comment says it bounds one page, no longer the whole listing.

`callTimeout` stays a const;
its comment says it bounds each request, and that githubclient applies its own client timeout from the end of a pause hold.

`TestListOpenIssues_PaginatesAndExcludesPullRequests` gains a second case: the second page is held by a pause past what a shared deadline has left, and completes after resume.
The row calls `listOpenIssues` with a per-request budget of 200 ms.
It builds its client with `githubclient.NewWithBaseURL(server.URL, nil)`, with `t.Setenv("GH_TOKEN", ...)` and `t.Setenv("GITHUB_TOKEN", "")` as `githubclient`'s own pause test does, and installs it through the `NewGitHubClient` seam in place of `installGitHubClient`, whose plain go-github client has no githubclient transport and so never meets the pause gate.
It installs a pause gate whose record the handler makes live while it serves the first page.
The handler answers the first page after 170 ms, so a deadline shared by both pages has 30 ms left when page 2 starts.
The gate's `Sleep` sleeps 150 ms and then lifts the entry, as in `githubclient`'s pause test, and the handler answers page 2 after 60 ms.
With a fresh deadline per page, page 2 has its whole 200 ms after release, 140 ms of margin, and `listOpenIssues` returns issues 1 and 3.
Under the shared deadline, card 2's extension leaves page 2 only its 30 ms remainder after release, so the call fails with a deadline error.
The row also asserts that the gate's `Sleep` ran once, so it fails when page 2 goes out unheld.
The row thus pins this card's change and the hold it claims, not the transport's extension alone.
`ListIssueComments`, `CommentAndClose` and `fetchOpenPRHeads` get the same loop shape with no row of their own; the one row pins the shape.
The test already runs serially, since it swaps the package-level `NewGitHubClient` seam, which also keeps the process-wide gate safe.

`fetchOpenPRHeads` reaches the real GitHub API only, through `githubclient.New`, so it gets no test of its own;
its per-page deadline has the same shape as the selfreport loops, and the extension each page gets is pinned by card 2's transport test.

**ImpactSummary:** Four GitHub callers in two packages move from one deadline per call to one per request; budgets per request are unchanged, and no signature changes.

**Commit:** 3: github callers: a fresh deadline per request
