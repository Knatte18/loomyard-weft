# Card 6 — github-network-hold

**Edit:**
- `internal/githubclient#authRT.RoundTrip`
- `internal/githubclient#allowedImports`
- `internal/githubclient/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `internal/githubclient#TestAuthRT_WaitsWhileAPauseCoversTheProcess`

**Intent:**
Hold every GitHub call while a pause covers the calling process (D4), inside `internal/githubclient` as `PATTERN-github-auth` and the decision's constraint require.
`authRT.RoundTrip` is the outermost transport every GitHub request goes through; it calls `done := pauserecord.Network("github")` before it resolves a token and defers `done`, so the 401 replay inside the same round trip is not held twice.
`allowedImports` gains `github.com/Knatte18/loomyard/internal/pauserecord`, itself a leaf of the standard library and `internal/lock`, so the package stays a leaf.
An operator's own `lyx selfreport create` under a hub entry therefore waits and prints the gate's one line naming the pause and `lyx resume`.

Bound: as for card 5, a wait at entry only, a no-op with no gate installed.

Test: the new test (not parallel, since the gate is process-wide) installs a gate over a temp record with a live hub entry and a `Sleep` that signals each wake, sends a request through an `authRT` whose transport is an `httptest` server counting hits and whose token comes from `GH_TOKEN`, asserts no hit after the first wake and exactly one gate line, resumes the entry, and asserts the request then completes with one hit; with the entry gone a second request completes with no wake.
`doc.go` names the hold beside the Authorization header; the package's budget row grows by one.

**ImpactSummary:** One hold call in the client's single round trip; every GitHub caller gains it, with no change while no gate is installed.
