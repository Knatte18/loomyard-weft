---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-correctness
    location: internal/shuttleengine/run.go:106
    summary: "A second `lym shuttle agent` with the same name in a worktree, `agent` by default, gets every instruction the operator typed into the earlier, still-live agent appended to its prompt as carried-over operator instructions, plus a replacement notice"
    fix: "Keep the operator agent out of the carry-over. Add a field to `shuttleengine.Spec`, such as `NoCarryOver bool`, documented as a session whose role key names no predecessor. Make `Runner.start` skip `newestPredecessor` and `carriedInstructions` for a spec that sets it, so no carry-over block, carried-instruction record or `noticeCarriedInstructions` call is made. Set the field in `operatoragent.SessionSpec`. Add a row in `internal/shuttleengine` that starts a second unawaited spec with the field set, the same role, and a predecessor whose waiter pid is gone and whose turns hold an operator instruction, and asserts the new prompt holds no `## Operator instructions carried over` block. Add the field to the assertions of `TestSessionSpec`."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/shuttleengine/run.go:191
    summary: "An agent session the operator ends leaves a dead pane in its own window under remain-on-exit, its run record reads running for good, and every reed resume relaunches it, since its strand has no done-when list and nothing stops it; the `open` path it replaces closed its window and was dropped on resume"
    fix: "Give the operator agent the end of life `Runner.Open` gives an opened session. Add a `shuttleengine.Spec` field the agent's spec sets, such as `EndsWithSession bool`. Under it, `launch` chains the launch command and the resume command with a touch of an exit marker in the run directory, `lym reed remove --name` of the strand-name variable with `--detach`, and an exit, the way `Runner.Open` builds its `Cmd` and `ResumeCmd` through `shell.ForGOOS()`. `launch` also passes the marker as the strand's `DoneWhen`, so a resume drops a closed agent. Set the field in `operatoragent.SessionSpec`, assert it in `TestSessionSpec`, and add a row in `internal/shuttleengine` asserting that the add spec of such a start carries the marker in `DoneWhen` and the self-remove in `Cmd`."
  - id: F3
    severity: MEDIUM
    class: consistency
    origin: handler
    location: contracts/stencils/operatoragent/operatoragent-template-start.md:23
    summary: "The parent directive the prompt renders contradicts the stencil. In the prime the no-parent variant tells the agent to write its report and stop when blocked, though it has no report and the stencil says to wait for the operator. In a task worktree the parent variant tells it to ask the run's orch over creel and end its turn, since its run waits for the answer, though the stencil says the operator works with it in its pane."
    fix: "Add a Go-filled marker to the stencil, such as `{{.mode_rules}}`, the way the Discussion chair's and the raddle agent's interactive rules reach their stencils. Fill it in `operatoragent.RenderPrompt` with a rule taking precedence over the parent section. A question or a blocker goes to the operator in this pane, there is no report to write, and the parent's creel mail is the only thing the parent section governs. Keep the parent directive marker, which `PATTERN-stencil-ownership` requires. Update the stencil's header comment to list the new marker."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/shuttlecli/cli_test.go:63
    summary: "The name normalization and rule-first refusal of `lym shuttle open --role` and `roleFromFlag`, the board entry's first ask, have no test"
    fix: "Add rows to `TestCommand_Run_FlagValidation` for `openCmd`, with `--role my_agent` wanting the exact error `--role \"my_agent\" is no valid agent name` followed by the rule text. Add a row to `TestOpenCmd_ModelAliasEngine` with `--role` and the value `My Agent`, asserting the `OpenSpec.Role` the recorder engine receives is `my-agent`."
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/shuttlecli/agent.go:47
    summary: "The new `lym shuttle agent` verb has no test. Nothing fails if it maps a wrong geometry field into the inputs, drops the language skills, calls `Open` in place of `StartSession`, or mis-wires its `--name` or `--model` refusal."
    fix: "Add a test in `internal/shuttlecli/cli_test.go` that drives `agentCmd` on the fake runner seam the open tests use. One row runs `--name` with the value `My Agent` in a task worktree geometry and asserts the started spec's `Role` is `my-agent`, its `ConfigRole` is the agent role, and its prompt names the geometry's shortname, hub path, worktree and slug. A refusal row asserts `--name my_agent` fails with the rule-first message."
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/shuttlecli/agent.go:93
    summary: "Under a pause covering the task's slug the agent button blocks with no bound and shows nothing, because `StartSession` waits out the pause and the button waits with no timeout. `open` refused at once with a way forward."
    fix: "Before `c.runner.StartSession`, refuse a covered worktree the way `Runner.Open` does. Move the coverage check `Runner.Open` makes on `r.pause` into a `Runner` method, such as `RefuseCovered(wayForward string) error`. Call it from `Runner.Open` with `re-run lym shuttle open` and from the agent verb with `re-run lym shuttle agent`. State in the verb's `Long` text that it is refused under a pause."
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/operatoragent/spec_test.go:411
    summary: "`TestSessionSpec` asserts neither the parent directive nor the PATTERN directive, so a dropped or swapped `ParentName` or a PATTERN directive that never fills passes"
    fix: "In `TestSessionSpec`, assert that the task row's prompt holds the parent name `ly:orch` and the prime row's prompt holds the line `No parent is recorded for this run.`. Give one row a `WorktreeRoot` under `t.TempDir()` holding a minimal `PATTERN.md`, and assert its prompt carries the implementer directive."
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/shuttlecli/open.go:34
    summary: "The `open` Long text restates the role rule as a literal instead of using `agentname.RoleRule`, which the `agent` verb uses, so the two can drift"
    fix: "In `openCmd`'s `Long` string, replace the literal `lowercase letters, digits and dashes, starting with a letter` with a concatenation of `agentname.RoleRule`, as `agentCmd`'s `Long` does, and import `internal/agentname`."
  - id: F9
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/landing/summary.md#Merge danger
    summary: "The Merge danger blast radius names reed alone. The diff also changes what every `lym shuttle open --role` accepts and passes on, adds a role to agentconfig, and adds a shuttle verb and stencil."
    fix: "In the `## Merge danger` section of `_lym/landing/summary.md`, name the blast radius as reed, shuttle (`lym shuttle open --role` now rewrites and refuses names, and the new `lym shuttle agent`), agentconfig (the new `agent` role) and stencils (the new opening stencil)."
  - id: F10
    severity: NIT
    class: consistency
    origin: lens:code-affected-set
    location: docs/overview.md:2044
    summary: "The standalone verb list for shuttle leaves out the new `agent` verb, which the module entry at line 649 lists"
    fix: "In `docs/overview.md` line 2044, change `lym shuttle run|open|manifest|state|await` to `lym shuttle run|open|agent|manifest|state|await|await-forks`, matching the module entry."
---

### [BLOCKING] A second agent inherits the operator's typed instructions from the live first one

**Location:** `internal/shuttleengine/run.go:106`, reached from `internal/shuttlecli/agent.go:93`

**Issue:** `launch` records `PID: os.Getpid()`, the pid of the short-lived `lym shuttle agent` process, which exits once the start returns.
`runIsLive` then reads the first agent's run as not live while its session is still running, and `unfinished` reads it as true.
The next `lym shuttle agent` with the same role in the worktree finds it through `newestPredecessor(roleKey(spec))`.
The role is `agent` by default, so this happens on the button's second click.
`start` then appends `carryOverBlock(carried)` to the new prompt, with the turns the operator typed into the first agent, as instructions to "follow as the operator's own".
It also sends the replacement notice, though nothing was replaced: reed numbers the new strand beside the live one.
The PR-Review and sampler callers of `StartSession` are long-lived row processes that start a fresh session only once the old strand is gone, so they never meet this case.

### [MEDIUM] A closed agent leaves a corpse window and comes back on every reed resume

**Location:** `internal/shuttleengine/run.go:191`

**Issue:** An unawaited spec has no output files, so `launch` gives its strand a nil `DoneWhen`, and its launch line carries no self-remove.
When the operator ends the session, reed's remain-on-exit keeps the dead pane in its window.
`planResumeLaunches` relaunches every non-live, non-held strand with an empty done-when list, so each closed agent comes back on the next resume.
No caller ever calls `StopStrand` for it, so its run record reads running for good and `lym shuttle state` keeps listing it.
`StartSession`'s own contract says the caller owns the stop; the agent verb returns at once and owns nothing.
The `open` path the buttons ran before touched an exit marker, removed its strand and exited, so its window closed and a resume dropped it.

### [MEDIUM] The parent directive contradicts the agent's stencil

**Location:** `contracts/stencils/operatoragent/operatoragent-template-start.md:23`

**Issue:** The stencil says the operator works with the agent in its pane, and its Start section says to wait for the operator's message.
In the prime, `ParentName` is empty, so `parent-directive-none` renders "When you are blocked, write your report and stop", and this agent has no report.
In a task worktree the recorded parent, such as `ly:orch`, renders `parent-directive-parent`.
That variant tells the agent to ask the orch with `lym creel post --parent --expect-reply` and end its turn, "your run waits for the answer".
The operator who started the agent is left out of that path, and the run being waited on is not this agent's.
The focus asked whether `parentdirective.Directive` agrees with an agent the operator talks to in its own pane; it does not in either variant.

### [MEDIUM] The name normalization and refusal of `open --role` have no test

**Location:** `internal/shuttlecli/cli_test.go:63`

**Issue:** `roleFromFlag` and its use in `openCmd` carry the board entry's first ask: spaces become dashes, and a refusal names the rule first.
No test fails if either breaks.
The existing open rows pass only roles that are already valid.

### [MEDIUM] The `lym shuttle agent` verb has no test

**Location:** `internal/shuttlecli/agent.go:47`

**Issue:** Nothing tests the verb.
It maps the reed geometry into `operatoragent.Inputs`, loads the language skills, starts through `StartSession` and refuses a bad `--name` or `--model`.
`TestSessionSpec` covers the spec builder alone, and the parity row passes its own inputs, so a wrong field mapping in the verb goes unseen.

### [LOW] The agent button hangs silently under a pause

**Location:** `internal/shuttlecli/agent.go:93`

**Issue:** `StartSession` calls `waitWhileCovered`, which waits with no bound and writes its notices to stderr.
`runButtonScript` waits for the button's command with no timeout and shows output only on a non-zero exit.
So a click under a pause covering the slug shows nothing until the pause lifts, then starts the agent.
The `open` path refused at once with `re-run lym shuttle open`.

### [LOW] `TestSessionSpec` asserts neither the parent directive nor the PATTERN directive

**Location:** `internal/operatoragent/spec_test.go:411`

**Issue:** The prompt checks cover the hub, the path, the worktree, the place clause and the edit directive.
A swapped or dropped `ParentName` passes, and so does a PATTERN directive that never fills: both rows use worktree paths that do not exist.

### [LOW] `open`'s help restates the role rule as a literal

**Location:** `internal/shuttlecli/open.go:34`

**Issue:** `agentname.RoleRule` is the named statement of the rule, and `agentCmd` uses it.
`openCmd`'s `Long` text writes the same words out by hand, so a change to the grammar leaves one of them wrong.

### [LOW] The Merge danger blast radius names less than the diff can break

**Location:** `_lym/landing/summary.md#Merge danger`

**Issue:** The change description names the blast radius as reed alone.
The diff also routes every `lym shuttle open --role` through the new normalization and refusal, adds the `agent` role to agentconfig, and adds the `lym shuttle agent` verb and its stencil.
The `two-way` door itself holds: no record format, config meaning or outward push changes.

### [NIT] The standalone shuttle verb list lacks `agent`

**Location:** `docs/overview.md:2044`

**Issue:** The module entry lists `agent` among shuttle's verbs, and the later standalone list does not.
That list already lacked `await-forks` before this diff.

## Rejected

- No `runproc` registration for the agent session (affected-set): `PATTERN-child-processes` asks for registration of a run's processes. The operator's agent belongs to no run, and a beam stop or teardown that kills the worktree's tmux session ends its pane. F2 covers the session's end of life.
- The board entry's "which skills to load" in the stencil (scope): `PATTERN-stencil-ownership` puts role skills on the launch spec and forbids a stencil asking an agent to load one, so the constraint wins and the spec's config role carries them.
- The unbuilt "ask again" branch of the name rule (scope): the entry offers it as one of two choices, and the diff builds the other.
