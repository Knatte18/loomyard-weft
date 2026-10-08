---
verdict: APPROVED
findings:
  - id: F1
    severity: NIT
    class: consistency
    location: internal/reedcli/smoke_dotfill_test.go:482
    summary: CrossClientControl sizing comment says a 12-row pane cannot show every strand's output, but a measured capture shows both strands' markers; the second strand keeps about four content rows, not two
  - id: F2
    severity: NIT
    class: consistency
    location: internal/reedengine/io.go:142
    summary: SetWaitMark doc says the status line renders the wait mark, while reedengine's doc.go now says the status bar renders it on the strand's button
---

Range: `_lyx/loom/rework/` does not exist, so the range is `git diff $(git merge-base main HEAD)..HEAD` with `product.parent` = `main` from `_lyx/shed/reed-view/status.json`.

Verified on HEAD, each run by me:

- `go build ./...`, and `go vet` with no tag, `integration`, `tmux` and `llm`: clean.
- `go test ./...` and `go test -tags integration ./...`: green.
  One earlier full run failed `TestWatchLoop_FailuresNeverKillTheLoop` (`select-layout attempts = 3, want 4`) while other builds ran concurrently; it passed on six isolated reruns and on the clean full rerun.
  The test and `watchloop.go` are untouched by the diff, and the test waits on a wall-clock window with 50 ms of slack, so this is load sensitivity in an existing test, not a defect of this diff.
- `go test -tags tmux` over `reedengine`, `orchengine`, `shuttleengine/...`, `vscode`, `ideengine`, `idecli`, `hubgeom`: green.

## Focus 1 — `TestSmokeDotFill` / CrossClientControl

`go test -tags tmux ./internal/reedcli/ -run TestSmokeDotFill` passes on HEAD, all four steps including CrossClientControl.
To check the sizing comment I ran a throwaway copy of the tree (outside the repo) with a capture print after both attaches.
The toucher pane captured 12 rows: the pane is 12 rows as the comment says, two of them the status rows.
The ten window rows split into three title rows (`dfalpha`, `dfbeta`, `selvage`) and seven content rows: two for the first strand, four for the second, one for Selvage.
Both `DOTFILL-MARKER-ALPHA` and `DOTFILL-MARKER-BETA` were on screen in the toucher pane, and the observed pane (27 rows) held the dot-fill run under them.
The pane size in the comment matches the size the test sets; the per-strand description does not (F1).

## Focus 2 — `TestIntegrationDriverBootstrap`

`go test -tags tmux ./internal/loomcli/ -run TestIntegrationDriverBootstrap` passes on HEAD.
`Runner.start` in `internal/shuttleengine/run.go` types, in this order after `awaitStartup`: the color sequence (when the strand has a color and `ColorByCaller` is unset), then `loadSkillsThenPrompt`, which sends the skill-load message and then the prompt pointer.
`integrationWriteStubDriverScript` reads the first line and skips it when it starts with `/color`, appends the Stop event on the load message, and reads the pointer as the next line.
The script consumes the lines in the order shuttle types them, and also works for a strand with no color, since the `case` only skips when a `/color` line is there.
Its doc comment describes exactly that order.
Not a defect.

## Focus 3 — stale comments in `reedengine` and `burlercli`

- `lifecycle.go` carries no mention of the status-line template, `ValidateStatusLine` or `StatusLineText`; the boot-validation comment names `validateBootConfig` and the segment colors.
- `doc.go` states "Launch sets the pane options and boot marks the strand window; every attach pre-flight marks the window and re-asserts both pane options on every strand bound to a live pane, … Selvage never gets either pane option".
  The code agrees: `launchStrandLocked` (`spawn.go`) calls `setStrandPaneOptionsLocked`; `upLocked` and `Resume` call `markResolvedStrandWindowLocked` after Selvage exists; `AttachArgv` calls `markStrandWindowLocked` and `reassertStrandPaneOptionsLocked` before any chain guard; `reassertStrandPaneOptionsLocked` walks strands only.
- `internal/burlercli/wiring_test.go` no longer names `RepoName` or an eight-field count; `Geometry.RepoName` has no remaining reference outside `lyxcwd.Location` uses.
- A repo-wide search for `ValidateStatusLine`, `StatusLineText`, `tokenvocab`, `{{.waits}}` and `status_line` finds only the config-compat test rows, `yamlengine`'s generic test data and the `docs/benchmarks/` measurement records, which the plan keeps as they are.
- One leftover wording: F2.

## Per-card checks

- Card 6 (Delete group): the statusline files, the `tokenvocab` package and the verb are gone, `go build ./...` shows no importer, and `reed.yaml` files carrying `status_line` still load (`config_test.go` row).
- Card 5, 7, 8, 9, 10, 11, 14, 15: the code matches each card's intent on the points I read against it — `bar.go` formats, `bindings.go` argvs and the optional-verb probe, `pinsForContent` and the pin run after `select-layout` on both apply paths and in the chain, `keepZoomLocked` and its hook and chain entries, `withRevivalFirst` and `hubgeom/spawnorder.go`, the color step order in shuttle and the orch watcher, and `SeedKeybindings`' merge and write rules.
- Every spawn site the plan lists names its segment; `shedrecipe`'s generic `SingleLLM` entry and `lyx shuttle run` build specs with no segment, but the plan's cards and the shipped recipe do not cover them, so I do not raise them.
- Test economy: the added tests follow the cards' test lists; I found no new test duplicating an existing test's coverage, no per-helper test that a public-surface test already covers (`TestNextSessionID` is the plan's own prescribed table beside the `switchClientVia` table), and no added behavior without a covering test.
- `cmd/lyx/testdata/test-budget.yaml` rows agree with the code (the budget test passes).

### [NIT] CrossClientControl comment mis-describes what the 12-row pane shows

**Location:** `internal/reedcli/smoke_dotfill_test.go:482`
**Issue:** The comment says that after the two status rows and one title row per pane "each strand keeps about two content rows, enough for the first strand's marker but not every strand's output".
A capture of the toucher pane at this size shows the first strand with two content rows and the second with four, and the second strand's marker is on screen.
The stated reason for waiting on the first strand's marker only (the second strand does not fit) does not hold at this size.
**Fix:** Restate the sizing from the capture, for example ten window rows split into three title rows and seven content rows, and drop the claim that the second strand's output does not fit, or give the real reason for waiting on the first marker alone.

### [NIT] `SetWaitMark` doc still names the status line

**Location:** `internal/reedengine/io.go:142`
**Issue:** "which the status line renders and which die with the pane" describes the retired single-line status; the wait mark now renders on the strand's button in the two-line status bar, and `doc.go` was changed to say so.
**Fix:** Say "which the status bar renders on the strand's button", as `doc.go` does.
