# Card 2 — shell-guard-table

**Create:**
- `internal/shellguard/doc.go`
- `plan:internal/shellguard#Switch` -> `type Switch string`
- `plan:internal/shellguard#SwitchNone` -> `const SwitchNone Switch = ""`
- `plan:internal/shellguard#SwitchDenyPython` -> `const SwitchDenyPython Switch = "deny_python"`
- `plan:internal/shellguard#Row` -> `type Row struct`
- `plan:internal/shellguard#Rows` -> `func Rows(foreground string) []Row`
- `plan:internal/shellguard#forbiddenSteerChars` -> `const forbiddenSteerChars = "x"`
- `plan:internal/shellguard#forbiddenNoticeChars` -> `const forbiddenNoticeChars = "x"`
- `plan:internal/shellguard#checkTexts` -> `func checkTexts(rows []Row) error`
- `plan:internal/shellguard#init` -> `func init()`
- `plan:internal/shellguard#TestMain` -> `func TestMain(m *testing.M)`
- `plan:internal/shellguard#allowedImports` -> `var allowedImports []string`
- `plan:internal/shellguard#TestLeafInvariant_AllowlistOnly` -> `func TestLeafInvariant_AllowlistOnly(t *testing.T)`
- `plan:internal/shellguard#TestCheckTexts` -> `func TestCheckTexts(t *testing.T)`
- `plan:internal/shellguard#payloadDenied` -> `func payloadDenied(t *testing.T, row Row, command string) bool`
- `plan:internal/shellguard#TestRows_PatternsDenyAndAllow` -> `func TestRows_PatternsDenyAndAllow(t *testing.T)`
- `plan:internal/shuttleengine/claudeengine#bashForegroundPhrase` -> `var bashForegroundPhrase string`
- `plan:internal/shuttleengine/claudeengine#renderShellGuard` -> `func renderShellGuard(row shellguard.Row) standingDeny`
- `plan:internal/shuttleengine/claudeengine#shellGuardInstalled` -> `func shellGuardInstalled(sw shellguard.Switch, in denyInputs) bool`
- `plan:internal/shuttleengine/claudeengine#shellRow` -> `func shellRow(t *testing.T, name string) shellguard.Row`
- `plan:internal/shuttleengine/claudeengine#TestStandingDenies_RenderEveryShellGuardRow` -> `func TestStandingDenies_RenderEveryShellGuardRow(t *testing.T)`

**Move:**
- `internal/shuttleengine/claudeengine#commandPositionPattern`
- `internal/shuttleengine/claudeengine#pythonCommandPattern`
- `internal/shuttleengine/claudeengine#steerPythonDeny`
- `internal/shuttleengine/claudeengine#noticePythonDeny`
- `internal/shuttleengine/claudeengine#inPlaceEditCommandPattern`
- `internal/shuttleengine/claudeengine#inPlaceCommandWordsPattern`
- `internal/shuttleengine/claudeengine#steerInPlaceEditDeny`
- `internal/shuttleengine/claudeengine#noticeInPlaceEditDeny`
- `internal/shuttleengine/claudeengine#rmCommandPattern`
- `internal/shuttleengine/claudeengine#shellExpansionPattern`
- `internal/shuttleengine/claudeengine#steerComputedRmDeny`
- `internal/shuttleengine/claudeengine#noticeComputedRmDeny`
- `internal/shuttleengine/claudeengine#sleepCommandPattern`
- `internal/shuttleengine/claudeengine#steerSleepDeny`
- `internal/shuttleengine/claudeengine#noticeSleepDeny`
- `internal/shuttleengine/claudeengine#goSubcommandPattern`
- `internal/shuttleengine/claudeengine#goArgumentsPattern`
- `internal/shuttleengine/claudeengine#goWordPattern`
- `internal/shuttleengine/claudeengine#rawGoCommandPattern`

**Delete:**
- `internal/shuttleengine/claudeengine#steerRawGoDeny`
- `internal/shuttleengine/claudeengine#noticeRawGoDeny`
- `internal/shuttleengine/claudeengine#TestRawGoDenyHook_DeniesModuleWideAndTierRunsOnly`
- `internal/shuttleengine/claudeengine#TestPythonDenyHook_DeniesCommandPositionPythonOnly`
- `internal/shuttleengine/claudeengine#TestInPlaceEditDenyHook_DeniesInPlaceScriptEditsOnly`
- `internal/shuttleengine/claudeengine#TestComputedRmDenyHook_DeniesRmOnComputedPathsOnly`
- `internal/shuttleengine/claudeengine#TestSleepDenyHook_DeniesMachineSleepVerbsOnly`
- `internal/shuttleengine/claudeengine#denyHookCommand`
- `internal/shuttleengine/claudeengine#preToolUseOutput`

**Edit:**
- `internal/shuttleengine/claudeengine#standingDenies`
- `internal/shuttleengine/claudeengine#init`
- `internal/shuttleengine/claudeengine#TestBuildSettings_DenyToggleMatrix`
- `internal/shuttleengine/claudeengine#TestBuildSettings_AgentAndBashHooks`
- `internal/shuttleengine/claudeengine#TestBuildDenyNotice_SentenceContent`
- `internal/shuttleengine/claudeengine/doc.go`
- `PATTERN.md`
- `pattern/PATTERN-leaf-packages.md`
- `docs/overview.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/shuttleengine/claudeengine#bashMaxTimeoutMS`
- `internal/shuttleengine/claudeengine#denyJSON`
- `internal/shuttleengine/claudeengine#standingDeny`

**Intent:**
Create `internal/shellguard`, the one neutral table of every guard lyx installs against shell commands (D5), and make `claudeengine` render it as its PreToolUse Bash hooks and notice sentences (D6), with the settings document and notice unchanged byte for byte.

`shellguard`'s surface:
- `Switch` names the config switch a row hangs on: `SwitchNone` for a row installed for every run, `SwitchDenyPython` for the row `shuttle.yaml`'s python switch turns off.
- `Row` carries `Name`, `Patterns []string` (grep `-E` patterns, all of which must match the payload for the guard to deny), `Steer` (the deny reason shown to the agent), `Notice` (the system-prompt sentence), `Switch` and `Forms [][]string` (the literal command forms the row denies as whole token sequences, for the no-denied-recovery scan; empty for a pattern-shaped row).
- `Rows(foreground string) []Row` returns a fresh slice of the rows in table order: `python` (patterns `pythonCommandPattern`; switch `SwitchDenyPython`; forms `python`, `python3`), `raw-go` (`rawGoCommandPattern`; always), `in-place-edit` (`inPlaceEditCommandPattern`; always; forms `sed -i`, `perl -i`), `computed-rm` (`rmCommandPattern` then `shellExpansionPattern`; always), `machine-sleep` (`sleepCommandPattern`; always; forms `systemctl sleep`, `systemctl hibernate`, `systemctl suspend`, `systemctl hybrid-sleep`, `systemctl suspend-then-hibernate`).
  The raw-go row is the one text an engine fills: its steer is today's `steerRawGoDeny` with the fixed `with the Bash tool timeout set to 3600000` replaced by `foreground`, so `Gate rule: run lyx gate test in the foreground ` + foreground + `, never in the background, ...`, and its notice is the steer plus `.`; every other row's steer and notice are the moved constants.
- The package doc in `doc.go` states the payload contract as part of the table's interface: the patterns match a JSON payload whose shell command is a JSON string value under a `"command"` key, as Claude's PreToolUse payload is, and an engine renders the rows against a payload of that shape or the task that adds the engine changes the table and its contract.
- `checkTexts` returns an error naming the row when a steer holds a single quote, a double quote or a backslash, or a notice holds one of those or a line break; `init` panics on `checkTexts(Rows(""))`.
  The two character sets move from `claudeengine`'s `steerTextForbiddenChars` and `noticeTextForbiddenChars` into `forbiddenSteerChars` and `forbiddenNoticeChars`; `claudeengine` keeps its own two for its Claude rows.

The Move group relocates the pattern, steer and notice constants with their comments unchanged into `internal/shellguard/rows.go`, the file `Rows` lives in.
`steerRawGoDeny` and `noticeRawGoDeny` are deleted because `Rows` builds the raw-go text from `foreground`.
`shellguard` imports the standard library, `internal/agentname` and `internal/gateslot` only, the last two for the three environment-variable names `rawGoCommandPattern` names; `TestLeafInvariant_AllowlistOnly` pins that set with `scankit.AssertImportAllowlist`.

In `claudeengine`:
- `bashForegroundPhrase` is `"with the Bash tool timeout set to " + bashMaxTimeoutMS`, Claude's phrase for running a long command in the foreground.
- `renderShellGuard` wraps a row as today's Bash entry: a one-pattern row as `grep -Eq '<pattern>' && echo '<deny JSON>'; true`, a row of several patterns in today's `in=$(cat); { printf '%s' "$in" | grep -Eq '<p1>'; } && { printf '%s' "$in" | grep -Eq '<p2>'; } && echo '<deny JSON>'; true` form, with `denyJSON(row.Steer)`, matcher `Bash`, the row's steer and notice, and `installed` calling `shellGuardInstalled`.
- `shellGuardInstalled` answers true for `SwitchNone`, `in.cfg.ClaudeDenyPython` for `SwitchDenyPython`, and false for any other switch.
  `init` panics on a `shellguard` row whose switch `shellGuardInstalled` does not know, a fail-loud wiring guard: a switch the table adds can never leave its row silently uninstalled, and no row reaches the guard while the table holds only the two switches.
- `standingDenies` becomes the Claude rows in today's order (fork Agent deny, Agent deny, webster-in-fork guard, AskUserQuestion deny) followed by `renderShellGuard` over every `shellguard.Rows(bashForegroundPhrase)` row in table order; the interactive AskUserQuestion recorder stays appended last by `buildSettings`.
  The webster-in-fork guard stays a Claude row, since it keys on Claude's fork payload field `agent_id`.
- `init` keeps checking every rendered steer and notice, the raw-go text filled with `bashForegroundPhrase` included.

The five pattern tables of `hookcmd_integration_test.go` move into one table-driven `TestRows_PatternsDenyAndAllow` in `shellguard`, in an `integration`-tagged file, keeping every row: each row names the guard by `Name`, a Bash command and whether it is denied, and `payloadDenied` encodes the PreToolUse payload as the deleted `preToolUseOutput` did and reports denied when every pattern of the row matches under a real `grep -Eq` run through `sh`, so each pattern denies and allows the same commands as before.
`TestMain` runs the package's tests under `tmuxkit.Main` after `gitkit.HermeticGitEnv()`, as `claudeengine`'s does, since the package now carries an `integration` file that spawns processes.
`TestCheckTexts` plants a row with a quote, a backslash and a line break in its steer and in its notice and asserts `checkTexts` refuses each naming the row; the shipped rows passing is already pinned by `init` at package load.
`hookcmd_integration_test.go` keeps only the recording-hook test and its helpers, so its header comment is rewritten to say the file runs the recording hooks alone, the deny pattern tables now living in `shellguard`.
`TestStandingDenies_RenderEveryShellGuardRow` asserts every `shellguard.Rows(bashForegroundPhrase)` row has exactly one `standingDenies` entry with matcher `Bash`, its steer and notice, and the command shape above for its pattern count, in table order after the Claude rows (D6's pin that every row is rendered).
The tests that named the moved steer and notice constants look them up through `shellRow`, which returns the `shellguard.Rows(bashForegroundPhrase)` row with that name and fails the test when none has it.
`TestBuildSettings_MatchesGolden` from card 1 stays green unchanged, which is the byte-identical proof.

Docs and invariants in this card: `PATTERN.md` gains, under Agents and prompts, `` - `PATTERN-shell-guard-table` — Guarding a shell command: declare it only in `internal/shellguard`, and every engine renders every row; engine tool guards stay in their engine. ``; `pattern/PATTERN-leaf-packages.md` gains an `internal/shellguard` section naming its closed import set and saying the `gateslot` admission departs from the header's "never a feature package" for the two environment-variable constants `gateslot` owns, because declaring each name once outweighs a dependency-free leaf; `claudeengine`'s `doc.go` says its Bash denies render `shellguard`'s rows; `docs/overview.md` gains an `internal/shellguard/` line in the package tree.
`cmd/lyx/testdata/test-budget.yaml` gains an `internal/shellguard: 3` row; `claudeengine` loses five top-level tests and gains one, so its row stays.

**ImpactSummary:** Shell-guard constants leave `claudeengine` for the new leaf `internal/shellguard`; `claudeengine`'s settings and notice output is unchanged, five pattern tests move into one `shellguard` table test, and nothing outside `claudeengine` referenced the moved constants.
