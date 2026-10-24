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
- `internal/battencli#battenVerbTexts`
- `internal/battencli#Command`

**Extends:**
- `internal/reedengine/doc.go`

**Uses:**
- `internal/hubgeom#ParentOf`
- `internal/hubgeom#ReedGeometry`

**Intent:**
D8: `lyx batten run <slug> --window` opens its window in the reed session of the run's parent worktree, the prime's or a beam's, with the prime as its working directory, wherever the verb runs; a beam's session then holds only its members' windows.
`windowSite` resolves that worktree: `hubgeom.ParentOf(prime, slug)` names the parent; the prime for an empty answer, and the prime too when the named beam's worktree is absent here, since only a session present on this machine can hold the window.
`openWindowInPrimeSession` builds the reed engine over that site's config and `ReedGeometry` and opens through `OpenWindowIn` with the prime's anchor path as `dir`.
A beam with no orch has no reed session, and `OpenWindow` refuses a worktree with none (`ErrNoSession`), so for a beam site the opener first brings the beam's session up with the engine's `Up`, which creates it when absent and leaves a live one alone, before it opens the window; the prime site opens as today.
Bound: the session brought up holds the run's window and no orch strand, so `OrchLive` reads no orch in the beam, and `HubOrchs`'s readers, which take an orch only from a worktree whose orch state records a strand, find none there, so no notice, tender or hibernate read changes; the session lives until the beam's pair teardown ends it, the reed session being ended before any worktree is removed (PATTERN-pair-teardown), and an orch started later in the beam takes it as its own, since `lyx orch start`'s `Up` leaves a live session alone.
`windowSite` returns, beside the site, whether the opener brings its session up: true for a beam, false for the prime.
`Engine.OpenWindowIn` is `OpenWindow` with a working directory: a non-empty `dir` adds `-c <dir>` to the `new-window` call; `OpenWindow` keeps its signature and calls it with an empty `dir`, so its other callers are unchanged.
`TestWindowSite` rows: a member of a present beam opens in the beam with its session brought up, a task of the prime and a member whose beam worktree is absent open in the prime with no session brought up, and a resolver error is returned.
`TestOpenWindow` gains a row asserting `-c <dir>` reaches the multiplexer call for a non-empty `dir` and is absent for an empty one.
The reed package doc names the working directory option.
The run verb's help in `battenVerbTexts` and the `--window` flag help in `Command`, which say the prime's reed session, now name the run's parent worktree's reed session, the prime's or a beam's.

**ImpactSummary:** One new reed window entry point beside the old one, and batten's window opener picks the run's parent session; OpenWindow's existing callers are untouched.
