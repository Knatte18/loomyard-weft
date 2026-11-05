```yaml
lens: forced-edits
outcome: reported
```

Forced-edits review of the Slack relay plan: what each card's change forces outside the members it names.

- **BLOCKING** — card 10 (and card 5): no card registers the new `slack` module with the sandbox-coverage guard. `cmd/lym/sandbox_coverage_test.go:1` requires every registered lym module to be covered by a `**Covers:**` tag in a `tools/sandbox/*SUITE.md` file or named on the test's exclusion allowlist with a reason; `hibernate` sits on that allowlist at `cmd/lym/sandbox_coverage_test.go:46`. The new `slack` CLI module (card 10) and config module (card 5) get neither. Fix: card 10 lists the allowlist variable of `cmd/lym/sandbox_coverage_test.go`, adding a `slack` entry with its reason, or adds a suite `**Covers:**` scenario.
- **MEDIUM** — card 11, D6 resume pointer: `orchengine.ChooseStartPrompt` (`internal/orchengine/launch.go:42-50`) renders the resume pointer from `LastHandoff` that a relaunched orch session gets, and it never passes through `WithPresence`. Card 11 covers `ResumeContext`, the three `Watcher` sites and `orchCLI.launchOrch`, so a relaunch after a cycle while presence is `slack` gets a pointer with no presence line. Fix: add `internal/orchengine#ChooseStartPrompt`, or its caller with the `Paths` in hand, to card 11, with a row.
- **LOW** — card 11, `Paths` builders: `internal/orchcli/paths.go:16` `PrimePaths` and `internal/orchroute/paths.go:17` `orchroute.Paths` also build `orchengine.Paths`, and card 11 sets `SlackPresencePath` only in `orchPaths`. Any pointer rendered over the other two never carries the line. The Intent should say this is intended, or the field should be set in `orchroute.Paths`, the one constructor.
- **LOW** — card 4: `internal/hubgeom/doc.go:4-13` lists every teller (ReedGeometry, BurlerGeometry, WebsterGeometry, ReconcileGeometry, PauseGeometry, DobbyGeometry …). The new `SlackGeometry` leaves that list incomplete, yet card 4 has the file only under `**Uses:**`. Fix: move it to `**Extends:**` with one sentence for `SlackGeometry`.
- **LOW** — card 2: the `internal/lymdirs/dirs.go:1` file header lists the directory tokens it declares, and `SlackDirName` is not added to it. Name the header in card 2's Intent.
- **LOW** — card 10: `cmd/lym/seamsignature_test.go:13-31` and `:40-56` pin modules' `RunCLI` and `RunCLIIn` signatures. The lists do not hold every module (`creelcli` is absent), so this is not a guard failure. Still, `slackcli.RunCLI` and `RunCLIIn` would be pinned only by adding them, and no card does.
- **LOW** — card 3, row "program client in a non-orch strand's pane": end to end it depends on the daemon's Linux peer-credential walk (`resolveSender` returns early off Linux, `internal/creelengine/identity.go:52`). The new `TestCreelCLI` subtest has to be generated only on Linux, per `PATTERN-test-speed`'s "no test skips by design". The card does not say so.

Tool calls made beyond my context:
1. `lym quarry resolve` on the plan's existing `Edit`/`Uses` glyphs, to confirm each resolves. All did, apart from the `Create` handle `TestSlackGeometry`, which is expected.
2. A grep of `cmd/lym` tests, runproc role users, lymdirs users and docs for per-module lists, to find registrations the pre-read lacked.
3. `cmd/lym/seamsignature_test.go`, to see whether its seam lists are complete.
4. `internal/orchengine/launch.go` and the users of `Paths`, to check every pointer render site and `Paths` constructor.
5. Greps for module-name lists in tests, `fail`'s code mapping in `server.go`, and `hubgeom/doc.go`'s teller list, which the pre-read had not covered.
6. The `hubgeom_test.go` package and tier, and the `sandbox_coverage_test.go` header and its hibernate entry, to confirm the new test's file fit and find the coverage guard.
