---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: internal/adoptcli/doc.go:74
    summary: adoptcli doc says a pause still holds every process the adopt worker starts, an adopt check included, which the inverted gate no longer does
    fix: In internal/adoptcli/doc.go, reword the paragraph that ends "installs its own gate, which a pause still holds." A process the worker starts installs its own gate, which a pause holds only when it is a run's own process, meaning it carries the run-process mark, is a detached child or is a non-orch strand's agent. The dobby is marked through `beam dobby`; an adopt check is none of these and passes a pause.
  - id: F2
    severity: LOW
    class: design
    origin: lens:code-tests
    location: cmd/lym/main.go:157
    summary: no test drives the real root's pre-run into InstallNetworkGate, so dropping or mis-passing runCommand(cmd) there fails no test, though the entry asks for a test that a paused hub does not hold the operator verbs
    fix: In cmd/lym/main_test.go, add rows to TestRunCommand_MarksOnlyARunsOwnVerbs, or one test beside it, that runs the root's pre-run for an operator verb such as `board get` against a hub with a live hub entry and asserts pauserecord.GateCovered() is false, and for `shed run` that it is true. Reuse hubforge and the pause-record helper from internal/tendercli/cli_test.go, and keep the test in the lowest tier that fits.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/testkit/tmuxkit/tmuxkit.go:540
    summary: the tmuxkit server-environment filter holds the literal LYM_RUN_PROCESS and the test that pins it uses the same literal, so a rename of pauserecord.RunProcessEnv leaks the mark into panes with every test green
    fix: In internal/shuttleengines/shuttleengines_test.go, TestEnvScrub_MatchesTmuxkitServerEnviron, build the environ entry and the want-list case from pauserecord.RunProcessEnv instead of the literal "LYM_RUN_PROCESS". The kit keeps its literal, since it cannot import pauserecord, and the test then fails when the constant and the literal differ.
---

### [MEDIUM] adopt doc still says a pause holds every process the adopt worker starts

**Location:** internal/adoptcli/doc.go:74
**Issue:** The doc says a process the adopt worker starts, "a dobby or an adopt check among them", installs a gate that a pause still holds.
After this diff the gate holds only a run's own process: one that carries the run-process mark, a detached child or a non-orch strand's agent (`heldByPause`, `internal/tendercli/gate.go`).
Only `beam dobby` is annotated, so an adopt check now passes a pause and the sentence is false.

### [LOW] No test drives the real root's pre-run into the gate

**Location:** cmd/lym/main.go:157
**Issue:** `TestRunCommand_MarksOnlyARunsOwnVerbs` pins which commands read as a run's, and the `childTests` rows of `TestInstallNetworkGate_TellsThePairsSlugAndInstallsNothingOutsideAHub` pin what the gate does for each kind of process.
Nothing pins the join, the `runCommand(cmd)` argument passed in the root's `PersistentPreRun`.
The board entry asks for a test that a paused hub does not hold `board get`, `board list`, `machine name`, `selfreport create` and `lym s`, and the diff delivers it by composition only.

### [LOW] The tmuxkit literal and its pinning test share one string

**Location:** internal/testkit/tmuxkit/tmuxkit.go:540
**Issue:** The kit cannot import `pauserecord`, so it holds `"LYM_RUN_PROCESS"` as a literal.
`TestEnvScrub_MatchesTmuxkitServerEnviron` also spells the literal, so a rename of `pauserecord.RunProcessEnv` would leave the reed engine scrubbing the new name and the kit scrubbing the old one, and no test would fail.

## Rejected

- Lens code-tests F-T3 and the focus question on `internal/battenshed/innerrun_pause.go`, MEDIUM scope: the held line `lym s` printed, `...; it continues after "lym tender resume"...`, is written only by the gate waiter at `internal/pauserecord/hold.go:121`, while the innerrun hold reports `held by the pause of <slug> since <time>` without that tail.
  The innerrun hold runs only inside `shed run`/`step` and the dobby, which are a run's own processes and are meant to be held, so `lym s` (adopt, `beam start`, `reed attach`) passes a run-scoped pause once the gate is inverted. The entry's `lym s` case is met and the file needs no change.
- Lens code-tests F-T1, LOW: the mark set before the hub check is harmless, since the mark only records that the process is a run's own, a run command is a process that holds a gate in every hub it runs in, and tests set the variable through `t.Setenv`.
- Lens code-tests F-T4, NIT: the same drift as F3, folded into it.
- Lens code-correctness item 2, LOW: an unparsable `LYM_STRAND_NAME` reading as held is a deliberate fail-closed choice that `UnparsableStrandHeld` pins, and the lens reports it as an observation, not a defect.
- Test economy: `TestRunCommand_MarksOnlyARunsOwnVerbs` and `TestEnvScrub_MatchesTmuxkitServerEnviron` are unique in the redundancy report; `TestStripIdentityEnv_DropsNameParentAndRunMarkOnly` carries a `//testtiming:keep` whose reason names what it pins, which the covering tests do not assert; `TestInstallNetworkGate_TellsThePairsSlugAndInstallsNothingOutsideAHub` extends an existing table with rows for new branches and duplicates no other test. No test-economy finding.
- The held line printed at once by a verb that waits is existing waiter behaviour at `internal/pauserecord/hold.go`, and the diff leaves it unchanged, which satisfies the entry's bullet.
- The focus question on `SetGateExemptAll` callers (`lym adopt`, fabric teardown, `lym batten cancel`): they stay needed because an agent or a run-marked process can start those verbs, and the changed doc comments in `internal/pauserecord/doc.go` and `hold.go` match the behaviour. The one stale doc is F1.
