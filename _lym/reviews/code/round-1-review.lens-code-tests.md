```yaml
lens: code-tests
outcome: reported
```

Task: the code-tests lens, listing for each behaviour the diff adds or changes the test row that would fail if it broke.

**Per-behaviour list**

1. `TitleWithHandle` returns `<handle>: <name>` for a matching entry: pinned by `TestTitleWithHandle/AnEntryWithAHandleOpensTheName`.
2. `TitleWithHandle` returns the name alone when the board lacks the slug, the entry has no handle, the board is unreadable, or there is no board file: pinned by the other four `TestTitleWithHandle` rows.
3. The Warn log on an unreadable board: none. Stencil-style log text is not behavior a test needs to pin, so I raise no finding.
4. `statusWindowTitle` reads `location.WorktreeName` and `BoardDir(HubPath)`: pinned by `TestStatusWindowTitle` (handle, other slug, no board).
5. `ensureStatusStrand` passes `statusWindowTitle(...)` into `statusStrandAddSpec`: **none**. `TestStatusStrandAddSpec` only checks the helper's pass-through. If `ensureStatusStrand` reverted to `c.recipe`, no row would fail.
6. `statusStrandAddSpec` puts the title in `Display.Title`: pinned by `TestStatusStrandAddSpec`.
7. `runsWindowTitleIn` titles the prime `beam: prime` and a beam `P1: beam: <slug>` once the board names it, and falls back before that: pinned by `TestRunsWindowTitle_WiredFromTheWorktreeTheVerbRunsIn` (tier 1, two steps).
8. `ensureRunsStrand` calls `c.runsWindowTitle()` at add or replace time: **none**. `start_test.go` stubs the func to `beam: prime`, and the rows I could see do not assert the title reaches the `AddSpec`. This is a weak gap, since the title is only read lazily there.
9. The `{title}` placeholder is replaced and quoted: pinned by the new `TestCommandProducer` row "the title placeholder reaches sh as one quoted word".
10. `Title` is called only when `{title}` is present: **none**. The harness `Title` func always returns a value and never fails on being called, so the test cannot tell lazy from eager.
11. A hub-layer command that still holds `{slug}` keeps working: covered by the existing `{slug}` rows in `TestCommandProducer`. Unchanged behavior.
12. The template default `open_window_command` holds `{title}`: pinned by `TestLoadConfig` (`wantCommands`).
13. batten `commandDeps.Title` yields `<handle>: <slug>` or the bare slug: **none**. My grep of `internal/battencli/*_test.go` for `commandDeps`, `.Title()` and `Title:` found nothing. If `Title` in `wire.go` were wired to the wrong slug or board dir, or the `Title` field were left nil, no test would fail. A nil `Title` would panic at the first `{title}` use, which is the default command, so the default Open-Window row would crash.
14. `lym reed window` with no `--title` keeps the base name: unchanged, still pinned by `DefaultTitleIsTheBaseName` in `window_test.go`.
15. The `Command.Title` seam is listed in `intentionallyNil` for loom wiring: pinned by `wiring_seams_test.go`.
16. `boardkit.WriteBoard` is a helper exercised by its callers, so it needs no test of its own.

**Findings**

- **MEDIUM, design, `internal/battencli/wire.go:840`.** `commandDeps.Title` (`<handle>: <slug>` from `BoardDir(location.HubPath)` and the task slug) has no test of its own. The board entry asks tests to pin the titled and fallback names of the Open-Window window. `TestCommandProducer` only uses a stub `Title`, and `TitleWithHandle` is tested in isolation. Evidence: the empty grep of `internal/battencli/*_test.go`, plus the `Title: func() string { return "T4: slug-one" }` stub in `command_test.go`. Fix: add a test in `battencli` that builds `commandDeps` over a hub whose board holds the slug with handle `T4`, and asserts `Title()` is `T4: <slug>`, and the bare slug with no board.
- **LOW, design, `internal/loomcli/sharedbootstrap.go:301`.** Nothing pins that `ensureStatusStrand` feeds `statusWindowTitle` into the spec. A revert to `c.recipe` would pass every test. Evidence: `TestStatusStrandAddSpec` and `TestStatusWindowTitle` each call their helper directly. Fix: if the existing `ensureStatusStrand` tests have a fake reed capturing the `AddSpec`, add a row asserting `Display.Title` is the handle-opened title; otherwise record this as accepted.
- **LOW, design, `internal/shedadapters/command.go:120`.** Nothing pins that `Title` is called only when `{title}` is present. The harness `Title` func always succeeds and counts nothing. Evidence: `command_test.go` `Title: func() string {...}`. Fix: have the harness `Title` count its calls and assert zero on rows without `{title}`, as the existing `TaskDir` laziness is pinned, if it is.
- **LOW, design, `internal/orchcli/runs.go:80`.** Nothing pins that `ensureRunsStrand` reads `c.runsWindowTitle()` when it builds the spec. The wired-title test only calls the func directly. Fix: in the start harness, make the title func return a distinctive value and assert the spec passed to `AddStrand` or `ReplaceStrand` carries it.

**Tool calls**
1. `grep` in `internal/battencli/*_test.go` for `commandDeps`, `.Title()` and `Title:`: to check whether a test covers batten's `Title` wiring, which my context lacked.
