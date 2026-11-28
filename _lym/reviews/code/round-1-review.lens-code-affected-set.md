```yaml
lens: code-affected-set
outcome: reported
```

Task: the code-affected-set lens, covering what the diff forces outside its touched lines. I made no edits.

**Set per change**

| Change | Set found | Covered? |
|---|---|---|
| `beam.window_placement` key and lookup | `beamsetting` and `dobbyengine` comments, `doc.go`, template, `LoadConfig`/`CheckLocalValues`, board help, `PATTERN-config-layers` rule, tests | Covered |
| `desktop.window_placement` and `tiling.*` keys, new `LoadConfig` refusals | `configreg` classification test, template, `hubgeom/buttons.go`, `ideengine/spawn.go`, `AfterWindow` | Covered; naming issue in F2 |
| `runWindowCommand` context and process group | `AfterWindow` callers (`lym reed window`, `lym ide`), `AfterPlacement`, tests | `AfterWindow` callers changed (F3) |
| `termwindow` `Open`/`IsKonsole`/`--separate` | `OpenAttach` callers, termwindow tests, package doc | Covered |
| `openTerminal` signature (`reedcli`, `machinecli`) | both seams and their tests, `window.go` Long help, `docs/overview.md` | Covered; stand-in issue in F4 |
| New `place-window` verb | pre-run skip list, comments, `reedcli/doc.go`, cli-cobra form, overview reed verb list (it already omits launcher verbs) | Covered |
| `runproc.RolePlacement` | readers listed in card 12; the other role-filtered readers found by search | Covered |
| `internal/kwinplace` (new) | overview package tree, `PATTERN-lym-files`, tolerant records, child-process logging, test-only code, `doc.go` rule | F1, F5 |
| `lymdirs.WindowPlacesDirName` | `PlacesPath` only | Covered |
| `parentwait.go` unbounded wait | guard list, `PATTERN-daemon-wakeups` | Covered |
| `tier3` now includes a desktop session | speed table, `running-tests.md`, `loom-plan-spec.md`, prose in `PATTERN-test-speed` | Partly (F6) |
| Declared PATTERN changes | none declared in the directive | Nothing to check |
| Commit messages and departures | no `Pattern-Departure:` trailers | Nothing to judge |

**Findings**

**F1. BLOCKING, design: `Snapshot.ZoneOf` is called only by tests.**
- Location: `internal/kwinplace/snapshot.go:72`.
- This breaks `PATTERN-test-only-code`: a production function no `main` reaches. Production uses `zoneHolding` through `CurrentOrder`.
- Card 6 asks for the method, but the constraint wins.
- Evidence: the only `ZoneOf(` call is `internal/kwinplace/recordings_test.go:138`.

**F2. BLOCKING, consistency: the new config keys are repeated string literals, not named constants.**
- Location: `internal/desktophook/config.go`, in `ConfigSpec` and the new refusals in `LoadConfig`.
- `"window_placement"`, `"tiling.min_tile_width"` and `"tiling.min_tile_height"` are repeated, passed to `configengine.KeyName` as literals and typed again inside the `--set` text.
- `PATTERN-config-key-names` requires rendering "from named constants", and `PATTERN-code-conventions` requires one constant per name.
- The older `window_wait_ms` lines do the same, but the constraint outranks that convention.
- Evidence: diff piece 007, the `config.go` hunks.

**F3. MEDIUM, design: `AfterWindow`'s callers no longer behave as before.**
- Location: `internal/desktophook/window.go`, `runWindowCommand`.
- `ConfigureGroupTerminate` sets `Setpgid` for every caller, including `lym reed window` and `lym ide`, which pass `context.Background()` and handle no signals.
- A Ctrl-C, hangup or group kill aimed at the verb's process group used to reach the window command. Now it ends `lym` and leaves the command running, with its 30-second bound gone along with the parent.
- The timeout also changed from killing the shell to SIGTERM then SIGKILL of the whole group.
- Card 4 states these callers "see the same behaviour".
- Evidence: `internal/proc/procgroup_linux.go` (`configurePlatformGroupTerminate`) and the `afterWindow`/`runWindowCommand` hunk in diff 007. I reasoned this from the code and did not run it.

**F4. BLOCKING, design: the `machinecli` tests add a stand-in for `internal/kwinplace`.**
- Location: `internal/machinecli/start_test.go`, `newStartDeps` and `TestStart_HandsEachPlacedWindowToThePlacementChild`.
- `handPlacement` is filled in production by `kwinplace.Spawn`, and the terminal opener's `(pid, placing)` answer is `kwinplace.OpenWindow`'s. The tests hand-write both and never pass through to the real module.
- `PATTERN-test-stand-ins` says a seam another module fills in production is simulating that module, and "No change adds a stand-in this rule forbids".
- Card 14 asks for this shape, but the constraint wins.
- The `reedcli` seams are filled by `reedcli`'s own functions, so they are not flagged.
- Evidence: `internal/machinecli/cli.go` `productionDeps` and diff 025.

**F5. BLOCKING, consistency (a judgement call): `doc.go` states a rule other code must follow.**
- Location: `internal/kwinplace/doc.go`: "It owns the KWin and kscreen boundary and the placement flow: no other package reaches either."
- `PATTERN-code-conventions` says such a rule is a PATTERN entry, never a `doc.go` paragraph. No entry carries it.
- The fix inside this diff is to drop the "no other package" clause.
- Evidence: diff 009, `doc.go`.

**F6. LOW, consistency: two statements still describe `tier3` as LLM-only.**
- Locations: `contracts/specs/loom-plan-spec.md:346` ("Tier 3 tests drive a real LLM; no gate runs them.") and `pattern/PATTERN-test-speed.md`, "No gate runs `tier3`" ("because a `tier3` run is billed, nondeterministic and needs a live session").
- The diff widens the tier to desktop sessions, and its own table row says billing applies to an LLM only.
- Evidence: the grep and `sed` output of the spec, and the pattern file.

Outside my lens: `d168c0b36` adds an entry to the `allowedNonHermetic` allowlist in `cmd/lym/hermeticenv_test.go`, and no commit in the range carries a `Loosened-Check:` trailer.

**Tool calls**
1. Grep for per-task setting lists: the context did not show which other docs or specs enumerate these settings.
2. Grep for `tier3` wording and reed verb lists: neither set was in the manifest.
3. Grep for window-opening docs and state-directory docs: I had not read docs beyond the diffed hunks.
4. Grep for `ZoneOf` callers, plus the overview reed section and plan-spec lines: the earlier search output was truncated, and neither doc passage had been read.
