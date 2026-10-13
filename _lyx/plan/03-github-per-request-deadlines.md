# Card 3 — github-per-request-deadlines

**Edit:**
- `internal/selfreportengine#callTimeout`
- `internal/selfreportengine#ListOpenIssues`
- `internal/selfreportengine#ListIssueComments`
- `internal/selfreportengine#CommentAndClose`
- `internal/selfreportengine#TestListOpenIssues_PaginatesAndExcludesPullRequests`
- `internal/fabriccli#openPRListTimeout`
- `internal/fabriccli#fetchOpenPRHeads`

**Intent:**
A GitHub caller whose one context deadline spans several requests derives a fresh deadline per request instead, so each request's own extension from card 2 covers its own hold (D6).

- `ListOpenIssues` and `ListIssueComments` derive `context.WithTimeout(context.Background(), callTimeout)` inside their page loop, once per page, cancelled before the next page.
- `CommentAndClose` derives one deadline for the comment and a second for the close.
- `fetchOpenPRHeads` derives `context.WithTimeout(ctx, openPRListTimeout)` per page;
  `openPRListTimeout`'s comment says it bounds one page, no longer the whole listing.

`callTimeout` becomes a package `var`, so a test can shorten it;
its comment says it bounds each request, and that githubclient applies its own client timeout from the end of a pause hold.

`TestListOpenIssues_PaginatesAndExcludesPullRequests` gains a second case: the second page is held by a pause past the call timeout and completes after resume.
The test sets `callTimeout` to 50 ms and restores it, and installs a pause gate whose record the handler makes live while it serves the first page.
The handler answers the first page after 40 ms, so a deadline shared by both pages has 10 ms left when page 2 starts.
The gate's `Sleep` sleeps 150 ms and then lifts the entry, as in `githubclient`'s pause test, and the handler answers page 2 after 30 ms.
With a fresh deadline per page, page 2 has its whole 50 ms after release, and `ListOpenIssues` returns issues 1 and 3.
Under the shared deadline, card 2's extension leaves page 2 only its 10 ms remainder after release, so the call fails with a deadline error.
The row thus pins this card's change, not the transport's extension alone.
`ListIssueComments`, `CommentAndClose` and `fetchOpenPRHeads` get the same loop shape with no row of their own; the one row pins the shape.
The test already runs serially, since it swaps the package-level `NewGitHubClient` seam, which also keeps the process-wide gate safe.

`fetchOpenPRHeads` reaches the real GitHub API only, through `githubclient.New`, so it gets no test of its own;
its per-page deadline has the same shape as the selfreport loops, and the extension each page gets is pinned by card 2's transport test.

**ImpactSummary:** Four GitHub callers in two packages move from one deadline per call to one per request; budgets per request are unchanged, and no signature changes.

**Commit:** 3: github callers: a fresh deadline per request
