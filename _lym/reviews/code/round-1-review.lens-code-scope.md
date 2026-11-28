```yaml
lens: code-scope
outcome: reported
```

Task: trace every change in `e9a8aa10f..HEAD` to a decision, and every decision to a change (lens `code-scope`).

**Changes → decision**

| Change | Traces to |
|---|---|
| `query.js`, `apply.js`, recordings and their `source.yaml` | Added "first card runs the probes", added "two outputs", D3; card 1 |
| `apply.js` releases and resizes no window; recording tests assert tiled under the panel | Added "D6 clamp"; D6 |
| `beam.window_placement` key, values, lookup, refusals, template comment, tests | D7; card 2 |
| `boardcli` help names the new key; `PATTERN-config-layers` lists it | D7 and the Review criterion; card 2 |
| `boardcli` help rewords `depends_on` and `project` (`internal/boardcli/cli.go:577`) | Forced: I infer it trims bytes for the 2000-byte `Long` cap (`cmd/lym/clitree_test.go:169`); I did not measure the total |
| Desktop keys `window_placement` and `tiling.*`, their checks and tests | D5, D7; card 3 |
| `AfterPlacement`, context and process group in the window-command runner | D12; card 4 |
| `Launcher.Open`, `IsKonsole`, `--separate` | D1, D2; card 5 |
| `kwinplace` snapshot types, script fillers, readers, `kscreen.go` | D2, D3, D4; card 6 |
| `prelaunch.go` | D1, D12; card 7 |
| `runner.go`, runner and contract tests | D3, D10; card 8 |
| `tier3` covers a desktop session (`PATTERN-test-speed`, `running-tests.md`) | Forced by the contract test; card 8 |
| `hermeticenv_test.go` allowlist entry | Forced by the contract test's Konsole spawn |
| `places.go`, `lymdirs.WindowPlacesDirName` | D9; card 9 |
| `planner.go` | D4–D6, D8, D11; card 10 |
| `flow.go`, `parentwait.go` | D7–D10, D12; card 11 |
| `unboundedWaitFiles` entry, `PATTERN-daemon-wakeups` line | Forced by D8's wait; card 11 |
| `child.go`, `placewindowchild.go`, `RolePlacement`, pre-run skip, `reedcli/doc.go` | D10, D12; card 12 |
| `reed window` hand-off, Long help, `desktop` comments, `docs/overview.md` | D1, D12 and the Review criteria; card 13 |
| `machine start` hand-off, its doc and overview line | D12; card 14 |
| `kwinplace/doc.go`, `testmain_test.go` | Forced by the new package |

**Decision → changes**

| Decision | Built by |
|---|---|
| D1 | `prelaunch.go`, `Launcher.Open`, probe check and `off` record in `flow.go`, both window paths |
| D2 | `--separate`, returned pid, `Window.Matches` |
| D3 | `kwinplace` scripts, runner, readers |
| D4 | `kscreen.go`, output filter in `planner.go`, `sendClientToScreen` |
| D5 | Tile-size keys, `Capacity` |
| D6 | `GridFor`, `CurrentOrder`, `targetOrder`, `apply.js` grid |
| D7 | Both mode keys, `LookupWindowPlacement`, `Mode` |
| D8 | `Anchor`, `Plan`, `awaitParent`, fallback handling |
| D9 | `places.go`, `reportUpdateFrom` |
| D10 | `child.go`, hidden verb, locks, query cache, runproc registration |
| D11 | Planner names only target-desktop windows; a full desktop is never a target |
| D12 | Both hand-offs, `AfterPlacement`, SIGTERM context |
| Added: first card runs probes | Commit `bd8c3dfea` and the recordings |
| Added: probes on one output | Superseded by the next entry |
| Added: probes on DP-2 and HDMI-A-1 | Recordings, including `neve-kscreen-laptop-off`; `.scratch/kwin-probes/run-probes.sh` also builds an HDMI-A-1 grid, outside the diff |
| Added: D6 clamp | `apply.js`, recordings, tests |

**Findings:** none. No change is unrequested, and every decision is built. This lens judged only the trace, not whether each build is correct.

Outside my lens: `reportUpdateFrom` (`internal/kwinplace/flow.go`) computes current order over every window on a desktop, including minimized and other-output ones; the correctness reviewer should check this against D4 and D6.

**Tool calls**
1. Searched `cmd/lym/clitree_test.go` for the help cap: the manifest lacked its value.
2. Searched `.scratch/kwin-probes/run-probes.sh` for the outputs it places on: the script was not read.
3. Measured the upsert help text in `internal/boardcli/cli.go`: only the diff hunk was in context. The count stopped at the first string concatenation, so it settles nothing.
