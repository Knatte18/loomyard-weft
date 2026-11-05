---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: decision
    origin: handler
    location: internal/shuttleengine/claudeengine/scope.go:83-84
    summary: "Interactive sessions keep memory, ListAgents and every skill of their plugins, but the Goal limits memory to the orch and ListAgents to spawned roles"
    fix: "Leave the code in place and defer this finding to the run's parent. The only sound change is recording or refusing the correction with lym loom decision add, which lies outside the fixer's write surface. Do not revert resolveSessionScope."
  - id: F2
    severity: MEDIUM
    class: decision
    origin: handler
    location: internal/shuttleengine/claudeengine/scope.go:127-129
    summary: "ListAgents is denied through permissions.deny in the settings, where the Goal asks for --disallowedTools ListAgents on the launch line"
    fix: "Leave the code in place and defer this finding to the run's parent, who records or refuses the mechanism with lym loom decision add. The change description names it as a design decision and no Added after Discussion entry covers it."
  - id: F3
    severity: MEDIUM
    class: decision
    origin: handler
    location: internal/reedengine/template_posix.yaml:70
    summary: "The board button now preloads scribe:prose and scribe:conversation, which the Goal and its Added after Discussion entry do not ask for"
    fix: "Leave the code in place and defer this finding to the run's parent. The change description names the preload as the writer's own addition, and the parent records or refuses it with lym loom decision add. The same edit sits in template_windows.yaml and reedengine/config_test.go."
  - id: F4
    severity: MEDIUM
    class: decision
    origin: handler
    location: internal/shuttleengine/claudeengine/command.go:450-457
    summary: "An opened session outside a hub writes no settings and gets no scope, though the Goal scopes every session"
    fix: "Leave the code in place and defer this finding to the run's parent. The change description names the gap, and the parent records or refuses it with lym loom decision add."
  - id: F5
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/shuttleengine/claudeengine/scope.go:136-145
    summary: "The degrade path of sessionScopeFor, an unreadable plugins file, has no test"
    fix: "Add one row to TestSessionScope_ReachesWrittenSettings in scope_test.go. The row sets engine.claudeDir to a directory holding a malformed plugins/installed_plugins.json, calls Prepare, and asserts disableClaudeAiConnectors and disableBundledSkills are true and enabledPlugins is absent."
  - id: F6
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/shuttleengine/claudeengine/scope.go:160-170
    summary: "claudeConfigDir has no test, so the CLAUDE_CONFIG_DIR override and the home-directory default are unpinned"
    fix: "Add a table test for claudeConfigDir in scope_test.go with one row that sets CLAUDE_CONFIG_DIR and one that clears it. The test uses t.Setenv, so it is not parallel, and a comment says why. If claudeHomeDir cannot be redirected, test the override row alone."
  - id: F7
    severity: NIT
    class: scope
    origin: lens:code-tests
    location: internal/shuttleengine/claudeengine/scope.go:203-214
    summary: "readInstalledPlugins branches for an empty installPath, a file entry under skills and two installs of one plugin are untested"
    fix: "Add those three cases as rows of TestReadInstalledPlugins in scope_test.go rather than as a new test."
---

### [MEDIUM] Interactive sessions keep memory, ListAgents and all plugin skills

**Location:** internal/shuttleengine/claudeengine/scope.go:83-84, 104, 126-130
**Issue:** The Goal turns auto-memory off for spawned roles, with the orch alone keeping it, and denies `ListAgents` for spawned roles.
It also limits scribe skills to those a role preloads.
`resolveSessionScope` treats every interactive session alike, so PR-Review (`internal/landingshed/prsession.go:153`) and the sampler (`internal/samplerengine/spec.go:100`) keep memory, `ListAgents` and every skill of each plugin they turn on.
The orch keeps every scribe skill, not just the four it preloads.
`_lym/landing/summary.md` names the interactive split and the orch's scribe skills as going beyond the record, but no `Added after Discussion` entry records either.
This is the unrecorded parent correction of the rubric, so it is deferred to the parent and not reverted.

### [MEDIUM] ListAgents is denied in the settings, not on the launch line

**Location:** internal/shuttleengine/claudeengine/scope.go:127-129
**Issue:** The Goal says `--disallowedTools ListAgents` for spawned roles.
The diff writes `permissions.deny: ["ListAgents"]` into the settings instead, and `command.go` adds no flag.
`_lym/landing/summary.md` gives the reason, one document and no second flag on the resume line, but the record does not carry the change.
The deny reaches an autonomous run only, as the Goal asks, and `TestSessionScope_WrittenSettingsPerRole` pins it, so the behavior is right.
Only the mechanism departs from the record, which is the parent's to settle.

### [MEDIUM] Board button preload of scribe skills is unlicensed

**Location:** internal/reedengine/template_posix.yaml:70
**Issue:** The Goal turns `ly@loomyard` on for the board session and limits scribe skills to those a stencil preloads.
The diff adds `--skill scribe:prose --skill scribe:conversation` to the button in both reed templates and in `config_test.go`.
With plugins following preloads, those two skills also turn the whole `scribe@scribe` plugin on for the board session, as an interactive session keeps every skill of a plugin it has on.
`_lym/landing/summary.md` names the preload as the writer's own addition, so the parent records or refuses it.
The reedengine edit is consistent across the three files that carry the command, so the code needs no further change.

### [MEDIUM] No scope for an opened session outside a hub

**Location:** internal/shuttleengine/claudeengine/command.go:450-457
**Issue:** `OpenLaunch` writes settings only when the engine has a hub board directory, so an opened session outside a hub carries no scope.
The Goal reads "off for every session" for connectors, the `anthropic-skills` listing and bundled skills.
`_lym/landing/summary.md` and `doc.go` state the exception, but no `Added after Discussion` entry records it.

### [LOW] Unreadable plugins file is not tested

**Location:** internal/shuttleengine/claudeengine/scope.go:136-145
**Issue:** `sessionScopeFor` logs a read error and still turns off connectors, bundled skills and `skills_off`.
No test reaches that branch, since every `Prepare` and `OpenLaunch` test sets `engine.claudeDir` to a readable directory.
A change that returned the zero scope on error would leave connectors and bundled skills on with every test green.
The fail-open on `enabledPlugins` is the documented design and is not raised again here.

### [LOW] claudeConfigDir is not tested

**Location:** internal/shuttleengine/claudeengine/scope.go:160-170
**Issue:** Every test sets `engine.claudeDir`, so the `CLAUDE_CONFIG_DIR` override and the `~/.claude` default are never run.

### [NIT] readInstalledPlugins edge branches are not tested

**Location:** internal/shuttleengine/claudeengine/scope.go:203-214
**Issue:** The empty `installPath` skip, the non-directory entry skip and the merge of several installs of one plugin have no row in `TestReadInstalledPlugins`.

## Rejected

- Lens code-correctness F1, a symlinked skill directory missing from `plugin.skills`: speculative, since nothing read shows a plugin install that links its skill directories.
- Lens code-correctness F2, the `os.ReadDir` error on a skills directory dropped without a log: the doc comment of `readInstalledPlugins` states that a plugin without a readable skills directory has no skills, and a missing directory is the common case.
- Lens code-correctness F3, the fail-open on `enabledPlugins` when the plugins file is unreadable: it is logged and documented, no plugin id can be listed without that file, and the fail-closed alternative of refusing the spawn is a larger change than the Goal asks for.
- Lens code-affected-set F1, `docs/overview.md:435` omitting the new behavior: the lens read only a truncated slice and called it unverified, and the diff does change that row.
