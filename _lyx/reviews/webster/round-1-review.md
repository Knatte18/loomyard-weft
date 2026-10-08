---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: scope
    location: internal/reedcli/smoke_dotfill_test.go:484
    summary: TestSmokeDotFill/CrossClientControl fails under -tags tmux on this branch and passes on main; the 8-row toucher client no longer shows the first strand after card 5's two status rows and pane-border titles
  - id: F2
    severity: BLOCKING
    class: scope
    location: internal/loomcli/integration_driverbootstrap_test.go:173
    summary: TestIntegrationDriverBootstrap_ReturnsWithoutWaitingOnTheDriver fails under -tags tmux on this branch and passes on main; card 11's /color line is now the first typed line, and the stub driver still reads its first line as the skill-load message
  - id: F3
    severity: LOW
    class: consistency
    location: internal/reedengine/lifecycle.go:278
    summary: card 6 left two comments in lifecycle.go describing the deleted status-line template validation (the ensureServerAndSessionLocked doc and the comment on the healthy already-up path)
  - id: F4
    severity: NIT
    class: consistency
    location: internal/burlercli/wiring_test.go:33
    summary: header comment still lists RepoName among the reed geometry values and says TestReedGeometry asserts all eight fields, after card 6 removed Geometry.RepoName
  - id: F5
    severity: NIT
    class: consistency
    location: internal/reedengine/doc.go:63
    summary: doc says boot re-asserts all three strand options, but boot only sets @lyx_strands; only launch and the attach pre-flight set @strand and @strand_color
---

Review range: `_lyx/shed/reed-view/status.json` gives `product.parent: main`, and no `_lyx/loom/rework/` round exists, so the subject is `git diff $(git merge-base main HEAD)..HEAD` (commits 1–15).

Substrate run: `go build ./...`, `go vet` under `integration`, `tmux` and `llm`, `go test ./...` and `go test -tags integration ./...` all pass.
`go test -tags tmux ./...` fails in two packages.
Each failing test was rerun in isolation on HEAD (fails every time) and on a `git archive main` export (passes), so both failures come from this diff.

### [BLOCKING] Dot-fill cross-client control no longer renders its marker

**Location:** `internal/reedcli/smoke_dotfill_test.go:484` (`TestSmokeDotFill/CrossClientControl`, `tmux` tier)

**Issue:** The scenario attaches a client in a harness pane sized to about 8 rows and waits for `DOTFILL-MARKER-ALPHA`.
Its own comment says the pane is too short for the second strand, so it waits on the first strand.
With card 5's `status 2` and `pane-border-status top`, the client loses two rows to the bar and one title row per pane.
The capture shows the `dfalpha`, `dfbeta` and `selvage` title rows, the two bar lines, and no strand content, so the attach never sees the marker and the test times out.
Card 5 lists the `tmux`-tier tests whose row arithmetic assumed one status row, and this one is not among them.
The plan's `verify:` block does not run the `tmux` tier, so the gate did not catch it.

**Fix:** Adapt the scenario to the new geometry, for example by giving the toucher pane enough rows to show the first strand's content under the two status rows and the title rows.
Keep the scenario's premise that the toucher client is distinctly smaller than the observed one, and update the sizing comment to match.

### [BLOCKING] Driver-bootstrap stub reads the new color line as the skill-load message

**Location:** `internal/loomcli/integration_driverbootstrap_test.go:173` (`TestIntegrationDriverBootstrap_ReturnsWithoutWaitingOnTheDriver`, `tmux` tier); the cause is `internal/shuttleengine/run.go:475`.

**Issue:** Card 11 makes `Runner.start` type `/color <name>` after `awaitStartup` and before the skill load whenever the strand has a color.
The driver spec now names `segmentcolor.Coordinator`, and the fixture hub seeds reed's template with `coordinator: blue`, so the driver strand gets a color.
The stub in `integrationWriteStubDriverScript` reads its first line as the load message and its second as the prompt pointer.
It now takes `/color blue` as the load message and the load message as the pointer, finds no `prompt.md`, and never writes the drive report.
Card 11 changed the typed protocol this fixture models but left the fixture unchanged.

**Fix:** Have the stub consume the color line before the load message, and update its doc comment's description of the typed lines.
Alternatively, seed the fixture's reed config so the coordinator segment has no color, if this test should stay independent of the color step.
Either way the test should pass on the branch again.

### [LOW] Stale status-line comments in lifecycle.go

**Location:** `internal/reedengine/lifecycle.go:278` and `internal/reedengine/lifecycle.go:309`

**Issue:** Card 6 deletes `ValidateStatusLine` and the template, and the Delete-group check finds no surviving code callers.
Two comments still describe them, though.
`ensureServerAndSessionLocked`'s doc says it validates the "status-line template before any tmux round trip", and the healthy already-up branch says "The status-line template was already validated in the pre-tmux block above".
Neither is true any more.

**Fix:** Drop "status-line template" from the doc comment, which now validates through `validateBootConfig`.
Reword or remove the comment on the already-up branch.

### [NIT] burlercli wiring test header still names RepoName

**Location:** `internal/burlercli/wiring_test.go:33`

**Issue:** The header lists `SocketKey, SessionName, LogsDir, RepoName, and HubPath` as reed geometry values pinned by `TestReedGeometry`, and says that test asserts "all eight fields".
Card 6 removed `Geometry.RepoName` and its assertions from `standalonegeom_test.go`.

**Fix:** Remove `RepoName` from the list and correct the field count in the comment.

### [NIT] doc.go overstates where the strand options are re-asserted

**Location:** `internal/reedengine/doc.go:63`

**Issue:** The doc says "Launch sets the pane options; boot and every attach pre-flight re-assert all three".
`upLocked` and `Resume` call only `markResolvedStrandWindowLocked`, which sets `@lyx_strands`.
`reassertStrandPaneOptionsLocked` runs only in `AttachArgv`.
The code matches card 3; the doc does not.

**Fix:** Say that boot marks the strand window, and that the attach pre-flight marks it and re-asserts both pane options on every bound live strand.

## Focus items settled without a finding

- **Card 6:** `StatusLineText`, `ValidateStatusLine`, the `statusline` verb, `StatusLineConfig`/`status_line`, `internal/tokenvocab`, its `.gitattributes` line and `Geometry.RepoName` have no remaining code references, and `go build ./...` passes with the package gone.
  The only survivors are the comments in F3 and F4.
  `config_test.go` pins that an old `status_line` block still loads.
- **Card 10:** `runStartTime` reads the modification time of `shedrun.SeedFile(l, shedrun.SelfRunID)`.
  A missing or unreadable seed leaves the pair undated, and `sortSpawnOrder` places undated pairs after the dated ones, by worktree name, so the fallback is defined.
  The revival sentinel is raised only when no session is up and the predicate holds, and the re-run sets the skip, so revival never recurses.
- **Cards 7–9:** The bindings run from `pinGeometryOptionsLocked`, which runs at boot and in the attach pre-flight.
  The adjusted resize pins run after `select-layout` on both apply paths and in the attach chain.
  The optional-verb probe logs a warning and never fails.
  The zoom bracket wraps every step the plan lists.
- **Test budget:** For every package the diff touches, the row in `cmd/lyx/testdata/test-budget.yaml` equals its count of top-level test functions, excluding `TestMain` (`reedengine` 239, `reedcli` 36, `hubgeom` 10, `orchengine` 125, `shuttleengine` 145, `claudeengine` 92, `vscode` 12, `segmentcolor` 4).
  No new test function duplicates an existing one.
- **Cards 11–14:** Every spec builder the plan names sets the segment D13 assigns, including all four bouncer literals and the orch spec with `ColorByCaller`.
  The `segmentcolor` constant values match the `segment_colors` keys in both templates.
- **Card 15:** `mergeKeybindings` replaces an existing block in place, and a second run compares equal and reports `unchanged` without writing.
  Bytes outside the markers are kept, apart from the inserted lines.
  Both `testmain_test.go` files point `KeybindingsPath` at a temporary directory, and no other test reaches `Spawn`.
