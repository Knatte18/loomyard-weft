# Card 17 — refuse-raw-guard

**Create:**
- `internal/shellguard#refuseRawPattern`
- `internal/shuttleengine/claudeengine#standingDeniesFor`

**Edit:**
- `internal/shellguard#Rows` -> `func Rows(foreground string, refuseRaw []string) []Row`
- `internal/shellguard#init`
- `internal/shellguard#TestRows_PatternsDenyAndAllow`
- `internal/shellguard#TestCheckTexts`
- `internal/shuttleengine/claudeengine#shellGuardDenies` -> `func shellGuardDenies(refuseRaw []string) []standingDeny`
- `internal/shuttleengine/claudeengine#buildDenyNotice`
- `internal/shuttleengine/claudeengine#buildSettings`
- `internal/shuttleengine/claudeengine#init`
- `internal/shuttleengine/claudeengine#Claude.Prepare`
- `internal/shuttleengine/claudeengine#shellRow`
- `internal/shuttleengine/claudeengine#TestStandingDenies_RenderEveryShellGuardRow`
- `internal/shuttleengine/claudeengine#TestBuildSettings_NoForbiddenCharsInSteerText`
- `internal/shuttleengine/claudeengine#TestBuildDenyNotice_MatchesInstalledDenies`
- `internal/shuttleengine/claudeengine#TestPrepare_WritesArtifactsAndReturnsConsistentLaunch`
- `internal/shuttleengine/claudeengine/doc.go`
- `internal/shuttleengine#Config`
- `cmd/lyx#shellGuardForms`
- `contracts/specs/loom-plan-spec.md`

**Delete:**
- `internal/shuttleengine/claudeengine#standingDenies`

**Intent:**
Refuse a raw test command the hub lists in `refuse_raw` (D10), so every test run in an agent session goes through `lyx gate test`.

`shellguard.Rows(foreground, refuseRaw)` adds a `refuse-raw` row after `raw-go` when `refuseRaw` is non-empty, and renders no pattern for an empty list.
`refuseRawPattern` builds one grep -E pattern: `commandPositionPattern` followed by any listed prefix, its words joined by whitespace runs and the last followed by a word end, each word's `.` escaped; the loader already refused any word outside letters, digits, `.`, `_` and `-` (card 2), so no other metacharacter reaches the pattern.
Command position is the shell guard's own: the start of the command or after a separator, past an `env`, `exec`, `xargs`, `sudo` or `time` launcher.
Its steer and notice name `lyx gate test`, `--tier` and `--card`, and carry the gate rule's foreground phrase and its re-run-on-no-free-slot wording; the row is `SwitchNone` and its `Forms` are the listed prefixes' word sequences, so the no-denied-recovery scan sees them.
The `raw-go` row keeps its patterns; its steer names `--tier` in place of `--tags`.
Bound: a guardrail seeing static text only; it lets through a test behind a script or `bash -c`, a flag between the prefix's words such as `go -C dir test`, and an assignment prefix such as `GOFLAGS=x go test`.

`claudeengine` renders the row for every run: `shuttleengine.Config` gains `RefuseRaw []string` with no yaml key (`yaml:"-"`), a value no config file sets; `Claude.Prepare` loads `gate.yaml` through `gateslot.LoadConfig` over the engine's told dirs, the hub file inside a hub and the worktree's own file outside one, as the gate verb reads them, and sets the field on its copy of the config before it builds anything, so `buildSettings` and `buildDenyNotice`, whose signatures stay, iterate `standingDeniesFor(cfg.RefuseRaw)`, the Claude denies then every shell guard row, in place of the package-level `standingDenies`; an absent file (`ErrConfigAbsent`) leaves the list empty.
Any other gate config load error refuses the spawn with the loader's way forward and role clause, before any artifact is written.
The file's head comment and `buildSettings`' comment name `standingDeniesFor` in place of the table.
Settings render at spawn, so a running session keeps the list it started with.
The init check runs over the rows of a sample non-empty list, so a steer holding a forbidden character fails at load.

`cmd/lyx`'s `shellGuardForms` passes loomyard's `refuse_raw` value, `["go test"]`, so the scan bars every stencil and steer that offers `go test` as a way forward (`PATTERN-no-denied-recovery`).
The scan reads `contracts/specs` too, and three lines of `loom-plan-spec.md` still spell the form with no `never ` before it, so this card rewords them to keep its own gate green: the post-merge dependency paragraph names the `final` gate's whole run in place of `go build ./... && go test ./...`, and the `verify-module-wide` and `verify-package-missing` descriptions say "a raw Go test, build or vet command" in place of the spelled commands.
The webster, shed and loom Go literals that spelled the form, `publishFailureSteps`' raw steps, are gone since card 10, and the spec's verify examples since card 16.

Tests: `TestRows_PatternsDenyAndAllow` gains `refuse-raw` rows over `["go test", "dotnet test"]`: `go test ./internal/x`, `cd x && go test ./...`, `env go test ./x`, `dotnet test X.sln` denied; `lyx gate test --tier 0`, `grep -rn 'go test' internal`, `go -C dir test ./x`, `GOFLAGS=x go test ./x` and `go testing` allowed; and no row rendered for an empty list.
`TestCheckTexts` covers the new steer.
`TestStandingDenies_RenderEveryShellGuardRow`, `TestBuildSettings_NoForbiddenCharsInSteerText` and `TestBuildDenyNotice_MatchesInstalledDenies` render with a non-empty list; `TestPrepare_WritesArtifactsAndReturnsConsistentLaunch` asserts a seeded `gate.yaml`'s `refuse_raw` row reaches settings.json and the notice, gains a row where no `gate.yaml` exists and no `refuse-raw` row renders, and a row where an unparsable `gate.yaml` refuses the spawn with no artifact written.
`TestRows_PatternsDenyAndAllow`'s `raw-go` steer row asserts `--tier` and no `--tags`.

**ImpactSummary:** the shell guard table gains one config-driven row and its two callers pass the list; the package-level deny table becomes a function of it, read by the settings builders from one new non-yaml field of the shuttle config.
