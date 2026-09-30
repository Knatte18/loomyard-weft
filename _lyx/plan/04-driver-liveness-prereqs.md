# Card 4 — driver-liveness-prereqs

**Create:**
- `plan:internal/loomengine#LoomDriverStrandName` -> `const LoomDriverStrandName = "loom-driver"`
- `plan:internal/reedengine#ErrNoSession` -> `var ErrNoSession error`
- `internal/reedengine/nosession_integration_test.go`

**Edit:**
- `internal/loomcli#driverStrandDisplayName`
- `internal/reedengine#Engine.requireSessionLocked`

**Intent:**
Give batten the two things it needs to ask, from prime and without booting anything, whether a task worktree's loom driver is still running: the driver strand's name, and a recognisable "no session" answer from reed.

The strand name lives in `internal/loomcli` as an unexported constant, and `battencli` must not import another CLI package.
Declare `LoomDriverStrandName` in `internal/loomengine/driver.go`, carrying over the existing doc comment's reason the name is stable (reed's add has no upsert semantics, so every add and every lookup must use it).
Redefine `driverStrandDisplayName` in `internal/loomcli/bootstrap.go` as `loomengine.LoomDriverStrandName`, with its doc comment pointing at the engine constant, so every existing loomcli use and `driverTeardownCommand` keep their spelling.
That half is a pure relocation of a value: the existing bootstrap and driver-prompt tests cover it.

`Engine.Status` never boots a session, but on a worktree whose session is not up it returns `requireSessionLocked`'s plain `errors.New(noSessionMessage(...))`, which a caller cannot tell apart from a genuine read failure without matching message text.
Declare the exported sentinel `ErrNoSession` in `internal/reedengine/lifecycle.go` with `errors.New`, and have `requireSessionLocked` wrap it with `%w` on its no-session return only, keeping `noSessionMessage`'s text as the visible message.
The foreign-session refusal it returns first (`refuseLiveForeignSessionLocked`) and the `hasSession` probe error are not wrapped: neither says the session is absent.
Its doc comment names the one reason it is exported: a caller outside reed that reads `Status` as a liveness probe maps an absent session to "not live".

Test in `nosession_integration_test.go`, over `newColdScratchEngine`, the cold-engine fixture `ensuresession_integration_test.go` already uses: `Status` on an engine whose session was never started returns an error satisfying `errors.Is(err, ErrNoSession)` whose text still carries `noSessionMessage`'s wording.

**ImpactSummary:** One constant's value moves to loomengine behind a same-string loomcli alias, and reed's no-session error gains a wrapped sentinel with unchanged text; no caller's behavior changes.
