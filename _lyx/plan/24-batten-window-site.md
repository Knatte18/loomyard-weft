# Card 24 — batten-window-site

**Create:**
- `plan:internal/reedengine#Engine.OpenWindowIn` -> `func (e *Engine) OpenWindowIn(name, dir string, lyxArgs []string) (WindowResult, error)`
- `plan:internal/battencli#windowSite` -> `func windowSite(prime *lyxcwd.Location, slug string, parentOf func(slug string) (string, error)) (site *lyxcwd.Location, ensureSession bool, err error)`
- `plan:internal/battencli#TestWindowSite` -> `func TestWindowSite(t *testing.T)`

**Edit:**
- `internal/reedengine#Engine.OpenWindow`
- `internal/reedengine#TestOpenWindow`
- `internal/battencli#battenCLI.openWindowInPrimeSession`
- `internal/battencli#battenCLI.runInWindow`

**Extends:**
- `internal/reedengine/doc.go`

**Uses:**
- `plan:internal/hubgeom#ParentOf`
- `internal/hubgeom#ReedGeometry`

**Intent:**
D8: `lyx batten run <slug> --window` opens its window in the reed session of the run's parent worktree, the prime's or a beam's, with the prime as its working directory, wherever the verb runs; a beam's session then holds only its members' windows.
`windowSite` resolves that worktree: `hubgeom.ParentOf(prime, slug)` names the parent; the prime for an empty answer, and the prime too when the named beam's worktree is absent here, since only a session present on this machine can hold the window.
`openWindowInPrimeSession` builds the reed engine over that site's config and `ReedGeometry` and opens through `OpenWindowIn` with the prime's anchor path as `dir`.
A beam with no orch has no reed session, and `OpenWindow` refuses a worktree with none (`ErrNoSession`), so for a beam site the opener first brings the beam's session up with the engine's `Up`, which creates it when absent and leaves a live one alone, before it opens the window; the prime site opens as today.
`windowSite` returns, beside the site, whether the opener brings its session up: true for a beam, false for the prime.
`Engine.OpenWindowIn` is `OpenWindow` with a working directory: a non-empty `dir` adds `-c <dir>` to the `new-window` call; `OpenWindow` keeps its signature and calls it with an empty `dir`, so its other callers are unchanged.
`TestWindowSite` rows: a member of a present beam opens in the beam with its session brought up, a task of the prime and a member whose beam worktree is absent open in the prime with no session brought up, and a resolver error is returned.
`TestOpenWindow` gains a row asserting `-c <dir>` reaches the multiplexer call for a non-empty `dir` and is absent for an empty one.
The reed package doc names the working directory option.

**ImpactSummary:** One new reed window entry point beside the old one, and batten's window opener picks the run's parent session; OpenWindow's existing callers are untouched.
