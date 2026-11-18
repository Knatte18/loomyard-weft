---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    origin: assist:assist
    location: internal/shuttlecli/hookpython.go#hookPythonAllowedCmd
    summary: The new verb reports its failure as plain text on stderr, against PATTERN-cli-cobra's rule that errors are JSON via internal/output.
    fix: >-
      In hookPythonAllowedCmd's RunE, replace fmt.Fprintln(cmd.ErrOrStderr(), "shuttle: "+err.Error()) with output.Err(cmd.ErrOrStderr(), "shuttle: "+err.Error()) from internal/output, keeping exit 1 through clihelp.Abort and nothing on stdout. Update the Long text's "a failure goes to standard error" only if it no longer holds. In TestHookPythonAllowedVerb_FollowsTheHubsDenyPython, make the OutsideAWorktree row assert the JSON error object that output.Err writes, not the bare "shuttle: " prefix.
  - id: F2
    severity: MEDIUM
    class: design
    origin: handler
    location: internal/shuttlecli/hookpython.go#pythonAllowed
    summary: In a standalone run the hook reads the target repo's config or the template, while the launch read the state dir's config, so a standalone run set to deny_python false now has Python denied.
    fix: >-
      Make the verb read the config the launch read. Either have each engine render the launch's config dir into the python check command, as an absolute, shell-quoted argument (for example `lym shuttle hook-python-allowed --config-anchor <dir>`), and have pythonAllowed load shuttleengine.LoadConfig(configengine.Dirs{Anchor: dir}, "shuttle") when that argument names a standalone state dir; or have the verb fall back to the standalone state dir that standalonestate derives for the target when lymcwd.ResolveWithin finds no hub. Add a row to TestHookPythonAllowedVerb_FollowsTheHubsDenyPython that sets deny_python false in a standalone config and expects exit 0.
  - id: F3
    severity: MEDIUM
    class: scope
    origin: lens:code-scope
    location: internal/orchcli/cli.go:223
    summary: The board entry's orch-launch test is missing; the amendment drops only its hook-row half, and no test drives deny_python false at the hub default layer through the orch launch to a notice without the python sentence.
    fix: Add a test in internal/orchcli, at the lowest tier that fits, over a hubforge hub with deny_python false at the default layer. It goes through the orch launch's own config wiring (shuttleengine.LoadConfig(fabricengine.ConfigDirs(location), "shuttle") into the engine's Prepare) and asserts that the prepared launch's deny notice holds no python sentence while the python hook row stays installed. If that is not feasible, defer the narrowing to the parent with a lym loom decision add entry.
  - id: F4
    severity: MEDIUM
    class: consistency
    origin: lens:code-tests
    location: internal/shuttleengine/claudeengine/settings_test.go#TestBuildSettings_DenyToggleMatrix
    summary: The redundancy report lists this test as contained by TestPrepare_WritesTheManifest. It pins what that test cannot see, each Bash deny's presence and the PreToolUse count across every interactive, fork and publish combination, yet it carries no testtiming:keep marker.
    fix: Directly above func TestBuildSettings_DenyToggleMatrix, add `//testtiming:keep pins each Bash deny's presence and the PreToolUse count across every interactive, fork and publish combination, which TestPrepare_WritesTheManifest checks for one autonomous spec`, separated from the doc comment by a `//` line as the other kept tests in the file are.
  - id: F5
    severity: MEDIUM
    class: consistency
    origin: lens:code-tests
    location: internal/shuttleengine/codexengine/guards_test.go#TestInstalledGuards_CoverEveryRow
    summary: The redundancy report lists this test as contained by TestPrepare_InstallsGuardsFromItsInputs. That test derives its expected guards from installedGuards itself, so only this test's literal row-name lists catch a dropped python row, yet it carries no testtiming:keep marker.
    fix: Directly above func TestInstalledGuards_CoverEveryRow, add `//testtiming:keep pins the literal installed row names per input, which TestPrepare_InstallsGuardsFromItsInputs derives from installedGuards itself`, separated from the doc comment by a `//` line.
  - id: F6
    severity: LOW
    class: design
    origin: assist:assist
    location: internal/shuttlecli/hookpython.go#hookPythonAllowedCmd
    summary: The verb lacks the skip-stencil-seed annotation, so every matching Python call runs the root pre-run's stencil-seed pass, which resolves geometry and can pull and commit to Bolt before the guard answers.
    fix: Add clihelp.SkipStencilSeedAnnotation set to clihelp.AnnotationEnabled to hookPythonAllowedCmd's Annotations map, beside the existing clihelp.AudienceAnnotation entry.
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/shellguard/rows.go:21
    summary: The SwitchDenyPython doc still says the shuttle config's python switch turns the row off, but the row is now installed in every run and the switch decides only the hook's live answer and the notice at launch.
    fix: Reword the SwitchDenyPython comment to say the row is installed for every run, its hook asks the hub's current deny_python on each matching call, and its notice follows deny_python at launch.
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/configcli/configcli.go:484
    summary: The lym config help still says deny_python false turns the Python guard off for a hub as hook and notice, but the hook now stays installed and lets Python through, and the notice drops only for sessions launched afterwards.
    fix: Change the help line to say that setting deny_python false lets Python through in running sessions at their next call and drops the notice from sessions launched afterwards. Keep the example command line as it is.
  - id: F9
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shuttlecli/hookpython_test.go:188
    summary: No row runs the verb from below the worktree's anchor, so a regression from lymcwd.ResolveWithin to the strict cwd gate, or a dropped pre-run skip, would pass every row.
    fix: Add a row to TestHookPythonAllowedVerb_FollowsTheHubsDenyPython that runs from a subdirectory created under hub.PrimeWorktree() with deny_python false and expects exit 0 with no output.
  - id: F10
    severity: NIT
    class: consistency
    origin: lens:code-correctness
    location: internal/shuttleengine/codexengine/guards.go:70
    summary: The installedGuards panic still says only that the row would stay uninstalled, though it now also fires for a switch missing from switchAnnounced.
    fix: Change the panic message's ending to "so the row would stay uninstalled or unannounced", matching the function's doc comment.
---

### [BLOCKING] The new verb's failure output is plain text, not internal/output JSON

**Location:** internal/shuttlecli/hookpython.go#hookPythonAllowedCmd

**Issue:** `PATTERN-cli-cobra`'s background states that errors are JSON via `internal/output`.
The new verb's failure branch writes `fmt.Fprintln(cmd.ErrOrStderr(), "shuttle: "+err.Error())`.
The sibling hidden verbs `hook-batching` and `hook-edit` carry the same plain-text line.
That is existing debt and does not license a new site.
No `Pattern-Departure:` trailer in the range names this.
The hook discards the verb's stderr, so the deny behaviour is unaffected,
but the rule binds every command.

### [MEDIUM] Standalone runs: the hook resolves a different config than the launch read

**Location:** internal/shuttlecli/hookpython.go#pythonAllowed

**Issue:** A standalone burler or webster launch loads its shuttle config from `configengine.Dirs{Anchor: res.StateDir}` (`internal/burlercli/wiring.go:184`, `internal/webstercli/wiring.go:292`).
It sets the pane cwd to the target (`internal/standalonegeom/reedgeom.go:45`).
Both hooks now run the verb from the session's directory.
The verb calls `lymcwd.ResolveWithin(target)` and `fabricengine.ConfigDirs`, which reads the target repo's own layers or the template, whose `deny_python` is true.
I built the branch's lym and ran the verb from a plain git repository outside any hub: it exits 1.
Before this change, a standalone run whose state-dir config set `deny_python: false` installed no python hook.
Now it installs the hook, and the hook denies every Python call.
The amended decision asks the hook to read "the hub's current deny_python".
A standalone run has no hub, so the change must keep its launch-time source.

### [MEDIUM] The board entry's orch-launch test is not present

**Location:** internal/orchcli/cli.go:223

**Issue:** The entry's Idea asks for a test in which `deny_python: false` at the default layer gives an orch launch with neither the notice nor the hook row.
The added decision overrides only the hook-row half.
The notice still follows the launch value, so the notice half of the test stands.
`internal/orchcli` and `internal/orchengine` hold no test that mentions `deny_python`, `DenyPython` or the deny notice.
The engine tests call `buildDenyNotice` and `denyNotice` with a config the test builds itself, so the hub-layer-to-orch-launch path the entry suspected is never exercised.
The change description calls the test "replaced" on the writer's own call.
No recorded entry narrows it, and the description does not present this as a parent correction.

### [MEDIUM] TestBuildSettings_DenyToggleMatrix needs its keep marker

**Location:** internal/shuttleengine/claudeengine/settings_test.go#TestBuildSettings_DenyToggleMatrix

**Issue:** The redundancy report lists it as contained by `TestPrepare_WritesTheManifest`.
That test asserts guard presence for one autonomous spec.
This test asserts each Bash deny's presence, the python row included, over every interactive, fork and publish combination, and pins the exact PreToolUse count.
It pins what the covering test cannot see, so per the rubric it is accepted with a `//testtiming:keep` marker.

### [MEDIUM] TestInstalledGuards_CoverEveryRow needs its keep marker

**Location:** internal/shuttleengine/codexengine/guards_test.go#TestInstalledGuards_CoverEveryRow

**Issue:** The redundancy report lists it as contained by `TestPrepare_InstallsGuardsFromItsInputs`.
That test builds its expected guards and notice through `installedGuards` and `denyNotice`, so a wrongly dropped row changes the expectation along with the result.
Only this test's literal row-name lists, the `SwitchesOff` row included, catch that drop.
It is accepted with a `//testtiming:keep` marker.

### [LOW] The verb runs the stencil-seed pass on every Python call

**Location:** internal/shuttlecli/hookpython.go#hookPythonAllowedCmd

**Issue:** The root `PersistentPreRunE` calls `seedStencils` (`cmd/lym/main.go:181`).
Unless the command carries `clihelp.SkipStencilSeedAnnotation`, it resolves geometry.
When seeding is due, it reaches a Bolt pull and commit (`cmd/lym/stencilseed.go:43-57`).
The new verb is a read-only predicate on the hook's hot path and reads no stencils.
`hook-batching` and `hook-edit` lack the annotation too, while `creelcli`'s hook verbs carry it.

### [LOW] The SwitchDenyPython doc is stale

**Location:** internal/shellguard/rows.go:21

**Issue:** It reads "marks the row the python switch of the shuttle config turns off".
The diff updated the matching claim in `internal/shellguard/doc.go` and left this one.

### [LOW] The lym config help still says false removes the hook

**Location:** internal/configcli/configcli.go:484

**Issue:** The help reads "Python guard off for a hub (hook and notice)".
After this change, `deny_python=false` leaves the hook installed and lets Python through on the next call.
The notice drops only for sessions launched afterwards.
The template comment and `docs/overview.md` now say so; this help does not.

### [LOW] No row runs the verb from below the anchor

**Location:** internal/shuttlecli/hookpython_test.go:188

**Issue:** Every row runs from the prime worktree's anchor or from outside any repository.
Both the `ResolveWithin` choice and the pre-run skip go unpinned, though the verb's Long text promises "anywhere under its anchored directory".

### [NIT] The codexengine panic message omits the announced map

**Location:** internal/shuttleengine/codexengine/guards.go:70

**Issue:** The function now panics when `switchAnnounced` lacks a switch.
The message still names only the uninstalled outcome.

## Rejected

- Assistant F1, that both python hooks must resolve lym through `selfexec.LymBinary`: rejected. That rule in `PATTERN-child-processes` guards a command that resolves the running binary's own path, so it never re-execs the test binary. A bare `lym` on PATH resolves no such path. It is the convention every hook in both engines follows, including the creel, batching and edit hooks, and none of them calls `selfexec`.
- Assistant F6, that the config-load failure and missing-key default have no rows: rejected. The load error shares the one error branch the `OutsideAWorktree` row pins. A missing key's template fill is `configengine` behaviour that this diff does not change.
- The scope lens's LOW decision finding, that the writer dropped half of the Idea's test on their own call: merged into F3, which carries the same omission and offers the parent-deferral route.
- Assistant F3 and F4 at BLOCKING: kept as F4 and F5 at MEDIUM. `PATTERN-test-speed` gives a contained test's keep to the reviewer's finding, so a missing marker before that finding is no rule violation.
- Assistant F5 at MEDIUM: kept as F6 at LOW. The seed pass runs only when deployed content is due, and the sibling hook verbs carry the same cost.
- Assistant F7: merged into F3.

## Focus departures

- "The original report: do the launch paths of `lym orch` and other roles hand the shuttle config loader the hub's layers": I confirmed only that the `lym orch` wiring calls `shuttleengine.LoadConfig(fabricengine.ConfigDirs(location), "shuttle")` (`internal/orchcli/cli.go:223`). I did not verify the change description's claim about the odd hub's 08:35 launch artifacts. They live in another hub's run directories, outside this review's diff.
