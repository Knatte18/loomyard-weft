---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:36
    summary: D2 says a present roles map replaces the template map whole, but the shuttle config declares no open map, so the loader merges the template's driver and webster entries back in
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:6
    summary: The Goal says the TTL is set when a session starts or resumes, but both lines are built once in Prepare and reed replays the stored resume line; no Decision states where role and config come from or that the TTL is frozen at Prepare
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:48
    summary: D4 adds a start-refusing guard over the whole configured set without stating what it blocks, that a bad entry for an unrelated role stops every role's spawn, or what it cannot touch
  - id: F4
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:54
    summary: D5 puts the TTL on the existing spawn log line, which lives in shuttleengine where the resolved value is not known; the Decision does not say who logs it, and the acceptance criterion names no way to observe it
  - id: F5
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md:94
    summary: Acceptance criteria do not cover D4's whole-set check, and do not name the empty, differently cased and non-string values that the exact-match rule refuses
  - id: F6
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:78
    summary: The open-role-set premise cites lyx reed add, but that verb takes a raw command and never reaches claudeengine, so its role cannot select a TTL
---

### [BLOCKING] D2's replace-whole map semantics need a declared open map

**Location:** `_lyx/discussion/decision-record.md:36` (D2), `:16` (Scope), `:91` (Acceptance criteria); `_lyx/discussion/support-log.md:33` (Q2)

**Issue:**
D2 states that a `claude_prompt_cache_ttl_roles` map present in the file replaces the template's map whole, never merges, so an operator drops a role by leaving it out.
The acceptance criterion "a map present in the file without `driver` gives the driver the default" depends on it.
That is not what the loader does today.
`shuttleengine.LoadConfig` calls `configengine.LoadOrTemplate` with no open maps, and `yamlengine.FillMissing` (`internal/yamlengine/fill.go`, `fillMapping`) descends into a present mapping and appends every template key the file lacks.
A file whose map omits `driver` gets `driver: 1h` appended in memory, so the driver cannot be dropped and the criterion fails.
The same gap bites the other direction: `lyx config reconcile` prunes file keys the template lacks, so an operator's `orch: 1h` entry, the very escape D6 and the open risks point to, is not durable.
Replace-whole semantics exist only for a key declared as an open map (`configreg.Module.OpenMaps`, `batcher.ConfigOpenMaps` as the precedent), passed to the module's `LoadOrTemplate` call and registered on the module in `configreg`.
Neither Scope nor any Decision names that, so the plan writer would build D2 as written and miss the acceptance criterion.

**Fix:**
State in D2 that `claude_prompt_cache_ttl_roles` is declared an open map: `shuttleengine` gains a `ConfigOpenMaps()` returning that key, `LoadConfig` passes it, and the `shuttle` entry in `configreg` carries it as `OpenMaps`.
Add the change to Scope's first bullet and to the doc list, and add an acceptance row that an entry for a role the template lacks survives load and an entry the template has is not restored when the file leaves it out.

### [MEDIUM] The Goal and D1/D3 leave the resolve point implicit

**Location:** `_lyx/discussion/decision-record.md:6` (Goal), `:27-29` (D1), `:40` (D3)

**Issue:**
The Goal says shuttle sets the TTL "when it starts or resumes a session".
In the code, `claudeengine.Prepare(runDir, spec, cfg)` is the one place that builds both lines: it gets the role from `spec.Role` and the config from `cfg`, and returns `Launch{Cmd, ResumeCmd}`; `shuttleengine.start` hands both strings to reed, which replays `ResumeCmd` verbatim on a rebuild.
Nothing builds a resume line later.
So the resume line resolves the same TTL as the launch line by construction, and a config edit reaches only runs started afterwards.
D1 states the verbatim replay but not that both lines come from one `Prepare` call with one resolved value, and the Goal's "resumes" reads as if a rebuild re-resolves.
A plan writer could add a second resolve site on the resume path, or a test that edits the config between launch and rebuild.

**Fix:**
In D1, state that the TTL is resolved once in `Prepare` from `spec.Role` and `cfg` and passed to both `buildLaunchCmd` and `buildResumeCmd`, so a rebuilt or resumed session runs at the TTL its run started with and a later config edit does not change it.
Reword the Goal to "when it starts a session, on both the launch and the resume line".

### [MEDIUM] D4's start-refusing guard states no bound

**Location:** `_lyx/discussion/decision-record.md:48-50` (D4)

**Issue:**
D4 introduces a refusal on every claude spawn and checks the whole set, so a bad entry for any role, including one that never runs, stops the spawn of every role: the driver, the orch and a mid-run burler or judge alike.
The Decision names the error content but not the reach.
It also does not say what the guard cannot touch.
Prepare is the only caller path, it runs before `AddStrand` and the run directory is removed on failure, so a live or rebuilt session is untouched; that bound is real but unwritten.
The rubric's attack-surface question asks the Decision to state what the guard can block and what bounds it.

**Fix:**
Add to D4: the refusal happens in `Prepare` before any pane exists, so it blocks only a new run (every role, since the set is checked whole, including one that starts mid-loom) and never an already started or rebuilt session; one edit to `shuttle.yaml` clears it, and the message names that file.

### [MEDIUM] D5 does not say where the TTL is logged

**Location:** `_lyx/discussion/decision-record.md:54-55` (D5), `:67` and `:95`

**Issue:**
The only existing line of that kind is `logger.Info("shuttle: run started", ..., "role", spec.Role, ...)` in `internal/shuttleengine/run.go`, and it already records the role.
It sits in `shuttleengine`, which never interprets the TTL; the resolved value exists only inside `claudeengine.Prepare`.
Putting it on that line needs a field on `Launch` that `shuttleengine` forwards, or a new line from `claudeengine`.
D5 and the Constraints name neither, and both fit the seam differently: a `Launch` field is `shuttleengine` code handling the value, a `claudeengine` line is not "the existing line".
The acceptance criterion "the spawn log line records the resolved TTL and role" gives a test no observable: the logger is process-global, so a capture through `logger.SetOutput` must be a non-parallel test that names that state under `PATTERN-test-speed`.

**Fix:**
Decide in D5 that `claudeengine.Prepare` logs one `Info` line through `internal/logger` carrying the role and the resolved TTL, which keeps the value inside the seam, and drop "existing" from D5, the Constraints and the Acceptance criteria.
Say in the criterion that a test captures the logger output and runs without `t.Parallel`, with a comment naming the global.

### [LOW] Acceptance criteria miss D4's whole-set check and value shapes

**Location:** `_lyx/discussion/decision-record.md:94`

**Issue:**
D4 says one bad entry is found on the first spawn of any role.
The criterion only says a bad value refuses "the run's start"; it does not require that a bad entry for a role other than the one starting refuses it.
It also does not list the values the exact-match rule refuses and that a YAML loader can produce: an empty string, a null entry, `1H` or `5M` in the wrong case, and a bare number, which `yaml.v3` hands to a string field as text.
These are rows of the same test, per `PATTERN-test-economy`, not new tests.

**Fix:**
Extend the criterion: the refusal also holds when the bad entry is for a role other than the one starting, and the refused values include an empty, null, differently cased and numeric one.

### [LOW] `lyx reed add` is not a path to the TTL map

**Location:** `_lyx/discussion/decision-record.md:78`; `_lyx/discussion/support-log.md:22` and `:36`

**Issue:**
The assumption that unknown role keys are accepted because "`lyx reed add` and `lyx shuttle run` take caller-chosen roles" is half wrong.
`lyx reed add` takes a raw `--cmd` and registers the strand directly; it never calls `claudeengine`, so its `--role` cannot select a TTL.
Only `lyx shuttle run --role` reaches the map with a caller-chosen role.
The conclusion (an open role set, no refusal on unknown keys) still holds.

**Fix:**
Replace the premise with `lyx shuttle run` in the Auto-mode assumptions, the support log's rejected alternative and Q5.
