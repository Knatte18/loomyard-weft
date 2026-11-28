---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-affected-set
    location: internal/beamcli/dobbydeps.go:171
    summary: The dobby's refusal post calls the key "the autonomy setting" instead of naming beam.autonomy in full through configengine's key-name helper, as PATTERN-config-key-names and the record's constraint require.
    fix: "In internal/beamcli/keynames.go add `// autonomyName returns the full name of the beam key that decides how far the orch settles a run's halts.` over `func autonomyName() string { return configengine.KeyName(beamsetting.Module, beamsetting.AutonomyKey) }`, beside pullRequestName (both imports are already there). In tellRefusal at internal/beamcli/dobbydeps.go:171 build the line with it, for example `fmt.Sprintf(\"the %s of run %s is refused, so its notices carry autonomy: %s until it is fixed: %v\", autonomyName(), slug, dobbyengine.AutonomyOff, refusal)`. Change refusalPrefix in internal/beamcli/dobbynoticestamp_integration_test.go to the new opening, \"the beam.autonomy of run task1 is refused, so its notices carry autonomy: false\"."
  - id: F2
    severity: MEDIUM
    class: consistency
    origin: handler
    location: _lym/landing/summary.md#Merge danger
    summary: The two-way door and the one-word blast radius "autonomy" name less than the diff can break. Once a layer or a run_settings holds unattended, every older build refuses the whole beam config, which stops each reader of beam.yaml on that build, not autonomy alone. That is a change to a config format another binary reads.
    fix: "Rewrite the `## Merge danger` section of _lym/landing/summary.md. Make the door one-way once a layer holds unattended: a revert, or a machine on an older build, refuses the beam config load (dobbyengine.LoadConfig, which the dobby, loom wiring and lym config beam run) wherever a worktree, the hub default or a task's run_settings holds unattended. Name that as the blast radius beside the halt-notice stamp and the board's beam-mode text. Say the door stays two-way only while no layer sets unattended, which the template comment asks the operator to wait for."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/beamcli/dobbynoticestamp_integration_test.go:124
    summary: No test pins that an episode ends at the run's next read that is not refused and that a later refusal is posted again. The summary and runAutonomyStamps' doc both claim this, and the BeamDobbyRefused row is the last read of its fresh deps.
    fix: "In TestHubDobbyDeps_RunNoticePostCarriesTheRunsAutonomy, after the three posts of a row with wantRefusal and on that row's same deps: rewrite task1's beam.yaml to `autonomy: false` and post a run notice through deps.RunNotices.Post (want: the line stamped `; autonomy: false`, no refusal). Then rewrite it to `autonomy: maybe` and post another run notice (want: the refusalPrefix mail, then the line stamped `; autonomy: false`). Each deps builds its own runAutonomyStamps, so this has to happen inside the refused row's loop body, not as a new row. Unverified: no way to make the post fail inside this hub was found, so the failed-post retry stays without a row unless the fixer finds one."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/loomcli/autonomy_integration_test.go:79
    summary: No test drives unattended through a reader that asks only whether autonomy is on. Reverting the .On() call in runAutonomy, the pawl Autonomous seam or the Slack orchAutonomous wiring to a comparison with AutonomyOn passes every test.
    fix: "Add the row `{name: \"the beam's autonomy unattended forces the task's keys\", beamFile: \"autonomy: unattended\\n\", forced: true}` to the table of TestRunAutonomy_ForcesTheRunsKeysOverARealHub in internal/loomcli/autonomy_integration_test.go. It runs runAutonomy through driftDeps, wire's ApplyAutonomy and autonomySettingLookup. The pawl seam (internal/loomcli/pawlwiring.go:154) and the Slack wiring (internal/slackcli/wiring.go:131) get no row through this test, and no wiring test reaching them was found."
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/hubgeom/autonomy.go#AutonomyWithRefusal
    summary: A read that fails for a reason other than a refusal comes back with a nil refusal, and tellRefusal ends the episode on any nil refusal. A run whose value stays mistyped is posted to the orch again after every transient prime or chain failure between two reads.
    fix: "Make AutonomyWithRefusal return `(level dobbyengine.AutonomyLevel, refusal error, read bool)`, where read reports that the lookup answered without any failure. In tellRefusal (internal/beamcli/dobbydeps.go) take that flag and end the episode, with delete(s.told, slug), only on a read that succeeded. A failure that is not a refusal leaves the episode as it is. Update stamp, hubgeom.Autonomy, the hubgeom doc.go line and the callers in internal/hubgeom/autonomy_integration_test.go, and reword the runAutonomyStamps doc to \"The episode ends at the run's next read that succeeds.\""
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/beamcli/dobbydeps.go:161
    summary: tellRefusal holds s.mu across s.post, a creel post. A slow or hung post blocks every concurrent stamp of every run's mails, runs whose values are fine included.
    fix: "In tellRefusal, take the lock only to read told[slug] and to delete or set it. Release it before s.post(line), and take it again to set s.told[slug] = true after a successful post. Accept that two concurrent first refusals of one slug may both post, or close that by marking the slug pending under the lock before the post and clearing the mark when the post fails."
  - id: F7
    severity: LOW
    class: decision
    origin: lens:code-correctness
    location: internal/orchcli/notify.go:99
    summary: The halt notices loom and batten post through NotifyParentHalt and NotifyRunFromHalt stamp through hubgeom.Autonomy and drop the refusal. A refused value is told to the orch only when the dobby builds one of its own stamped mails, so a run under beam.run_watch off whose halts come only from loom or batten is stamped false and the orch never learns why.
    fix: "The parent chooses between two options. Accept the gap: D1 names the dobby as the poster, and the halt is still relayed under false. Then add one sentence to the runAutonomyStamps doc saying a refusal is told only once the dobby stamps one of its own mails about the run. Or build the missing part: have the dobby notice a refused value on its own tick for each listed run, independent of whether it posts a mail, and post through the same runAutonomyStamps episode."
---

### [BLOCKING] The refusal post names the key informally

**Location:** `internal/beamcli/dobbydeps.go:171`

**Issue:** The new line opens "the autonomy setting of run %s is refused".
PATTERN-config-key-names, and the record's constraint under it, require every message to name a config key in full, `beam.autonomy`, through `configengine`'s key-name helper.
The appended refusal does carry the full name, but the post's own text refers to the key by a short name.
`internal/beamcli/keynames.go` already holds the package's helpers for this.

### [MEDIUM] The Merge danger call names less than the diff can break

**Location:** `_lym/landing/summary.md#Merge danger`

**Issue:** The door is `two-way` and the blast radius is just "autonomy".
The record's own Bound says an older binary refuses the beam config while a layer holds `unattended`.
`checkTaskSetting` runs inside `LoadConfig`'s check of the resolved file, so the refusal stops the whole beam config load on that build, not only the autonomy readers.
The change widens a config format that other binaries read, which the rubric names as grounds for a `two-way` finding.
After the operator sets the value, a revert carries the same failure.

### [MEDIUM] Nothing pins how a refusal episode ends

**Location:** `internal/beamcli/dobbynoticestamp_integration_test.go:124`

**Issue:** The `BeamDobbyRefused` row pins one refusal mail before the first stamped mail.
No row reads the run again after it, so nothing pins the claim that a read that is not refused ends the episode and a later refusal posts again.
Each row builds fresh deps, so the episode-end reads must run inside the refused row.

### [MEDIUM] No test reads unattended as on

**Location:** `internal/loomcli/autonomy_integration_test.go:79`

**Issue:** The board entry delivers "Every reader that asks only whether autonomy is on reads `true` and `unattended` as on."
`TestRunAutonomy_ForcesTheRunsKeysOverARealHub` has only `true` and `false` rows.
`internal/slackcli/hook_test.go` fakes `orchAutonomous` as a bool.
None of the `.On()` call sites fails a test if it is reverted.

### [LOW] A transient read failure ends the refusal episode

**Location:** `internal/hubgeom/autonomy.go#AutonomyWithRefusal`

**Issue:** `AutonomyWithRefusal` returns a nil refusal both for a successful read and for a prime or chain failure, and `tellRefusal` deletes `told[slug]` on any nil refusal.
`hubgeom`'s own `autonomyFailures` episode keeps both kinds of failure in one episode, so the two episode notions disagree.
A run whose value stays mistyped is posted again after each transient failure.

### [LOW] The stamp mutex is held across a creel post

**Location:** `internal/beamcli/dobbydeps.go:161`

**Issue:** `defer s.mu.Unlock()` keeps the lock through `s.post(line)`.
A hung post blocks every stamp of the dobby's gate-hold notices, run notices and watch lines for every run.

### [LOW] Halt notices from loom and batten never tell the orch of a refusal

**Location:** `internal/orchcli/notify.go:99`

**Issue:** `WithRunAutonomy` drops the refusal that `AutonomyWithRefusal` hands back.
Callers include `internal/battencli/notices.go:30,32`, `internal/loomcli/drivewiring.go:161` and `internal/loomcli/prepush.go:82`.
So a refused value is noticed only when the dobby stamps a mail of its own.
The halt still reaches the orch stamped `false`, the level at which it asks the operator, so the cost is a missing explanation rather than a skipped operator.

## Rejected

- code-affected-set F1 (MEDIUM, `internal/orchengine/prompt.go:57`, the unattended line's pointer to `role-autonomy.md`): rejected. The role texts are another part of the Sketch (D9) and outside this entry's Takes, so changing them here would be unlicensed scope. The line names the level as D1 asks. An `autonomy: unattended` stamp that no role text keys on falls back to the operator-asking behaviour. The change description states the gap accurately.
- code-affected-set F2 (LOW, `internal/heddleengine/glossary.md:27`): rejected for the same reason. The glossary entry for the new level is D9's, outside this entry's Takes.
- code-affected-set's note that an older binary's `hubreconcile` walk might rename `unattended` in a board `run_settings`: not kept as its own finding. That walk renames only values a move table or reinterpretation names, and an older binary has none for `unattended`. The older-binary risk it points at is F2.

Verdicts on the focus directives:

- Episodes (directive 1): a run that builds no stamped mail never opens an episode (F7). A dobby restart loses the in-memory episode and posts once more, which is acceptable. Runs and concurrent posters are keyed per slug under one mutex (F6). A failed post leaves the episode untold, as documented. A non-refusal failure ends the episode (F5). The tests pin only the first post (F3).
- Checking paths (directive 2): the lookup, `LoadConfig`, `lym config beam --set` (through `CheckLoad`), `CheckLocalValues`, `LookupWithSource` with its env expansion, and the board mode map all take the three levels through one `checkTaskSetting`. No reinterpretation touches the key's values. No Go reader outside the diff still compares to a boolean or to the text `true`.
- Unattended at the orch (directive 3): consistent with D1 and the entry's Delivers. The role-text gap is out of scope; see Rejected.
- Messages (directive 4): the refusal in `checkTaskSetting` names `beam.autonomy` through `keyName` and is reached by `TestAutonomyLevels_EveryCheckingPathTakesTheThreeLevels`. Setting `unattended` through `lym config beam --set` runs the same `LoadConfig` check that test passes. The dobby's post line fails the key-name rule (F1). PATTERN-refusal-way-forward binds webster, shed and loom, not dobbyengine; the record's constraint is met by the refusal test.
- Change description (directive 5): the What changes and Tests claims match the diff. The Merge danger call does not (F2).
- Tests (directive 6): no assertion is loosened; the removed `TestLoadConfig` row is replaced by stricter assertions. The readers that ask only whether autonomy is on are uncovered for `unattended` (F4), and so is the episode end (F3).
