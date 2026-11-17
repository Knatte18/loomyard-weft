---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/shuttleengine/codexengine/codexengine.go:123
    summary: Prepare hands the user-setup source the unresolved anchor as WorkDir but the symlink-resolved checkout root as RepoRoot, so the two disagree whenever the anchor path holds a symlink.
    fix: In Prepare, set UserSetupRequest.WorkDir to the symlink-resolved workDir that Prepare already computes a few lines above (the value passed to projectDocMaxBytes), not c.dirs.Anchor. Add a row to TestPrepare_UserSetup (or TestPrepare_InstructionFiles) that reaches the worktree through a symlinked anchor and asserts the request's WorkDir is the resolved path, so skillListing's upward walk meets RepoRoot and project-scope plugin installs match.
  - id: F2
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: docs/overview.md:709
    summary: The shuttle codex engine paragraph of the overview does not describe the launch behaviour this diff adds, only one clause was changed from prompt file to developer instructions.
    fix: In the codex engine paragraph of docs/overview.md, add what the launch now carries - the standing items as developer_instructions with the 128 KiB argument refusal, the subagent effort and max_subagent_threads cap, the user setup (global file, CLAUDE.md files above the root, skill listing, plugin session-start contexts bounded by plugin_hook_timeout), project_doc_max_bytes with AGENTS.override.md carrying AGENTS.md and the import-expanded CLAUDE.md, and the instruction-file pointers. Keep it to the facts a reader needs, matching the paragraph's existing density.
  - id: F3
    severity: MEDIUM
    class: scope
    origin: lens:code-tests
    location: internal/shuttleengine/claudeengine/usersetup.go:538
    summary: The parent's recorded handling of an unresolvable Claude config directory (drop only the parts that need it, keep project skills and above-root files, one warning, no error) has no test row.
    fix: Add a row to TestUserSetup_SkillListing or TestUserSetup_InstructionFiles in claudeengine/usersetup_test.go that leaves source.claudeDir empty and makes claudeConfigDir fail (t.Setenv CLAUDE_CONFIG_DIR to empty and HOME and USERPROFILE to empty, which means that row does not run in parallel). Assert no error, one warning naming CLAUDE_CONFIG_DIR, no global file and no plugin skill or context, and that a project skill under the repository root and a CLAUDE.md above the root are still returned.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/shuttleengine/codexengine/instructionfiles.go:594
    summary: A CLAUDE.md with imports at the working directory is named as unread when a heddle-rendered AGENTS.override.md sits beside it and no AGENTS.md exists, although that override already carries it expanded.
    fix: In claudeFileIsUnread, apply the heddleRendered exemption for the working directory before the import branch, not only inside the AGENTS.md branch, so a CLAUDE.md beside a heddle-marked AGENTS.override.md at workDir is never named. Add a TestPrepare_InstructionFiles row with a marked override, no AGENTS.md and a CLAUDE.md that imports a file, expecting no pointer.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/shuttleengine/codexengine/instructionfiles.go:565
    summary: unreadClaudeFiles walks the whole checkout on every Prepare, descending into git-ignored trees such as node_modules and build output, which costs time and lists vendored CLAUDE.md files as pointers.
    fix: In unreadClaudeFiles, skip directories that hold no tracked source, for example by skipping a closed list of dependency and scratch directory names next to .git, or by skipping a directory whose path git ignores through the existing gitexec client. Name the skipped set as one constant, and add a TestPrepare_InstructionFiles row with a CLAUDE.md inside a skipped directory expecting no pointer.
  - id: F6
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/shuttleengines/shuttleengines.go:82
    summary: Several added failure paths and wirings have no test row - the RunHook wiring of the registry, Root passed by lym heddle render, the examine-instruction-file refusal, and the plugin file, skill file, matcher and hook-type error paths.
    fix: Extend TestNew_CodexLaunchCarriesTheClaudeUserSetup with an installed plugin whose SessionStart hook prints a marker and assert the marker rides the launch line. Add a heddlecli render test asserting the request carries event.Root (a codex-file import is recorded). Add a TestPrepare_InstructionFiles row where an instruction file stat fails for a reason other than absence and the launch is refused naming it. Add rows to TestUserSetup_SkillListing and TestUserSetup_PluginSessionStartContexts for a plugin with no installPath, an unreadable plugins file, an invalid matcher regex and a non-command hook type.
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: internal/shuttleengine/codexengine/prepare_test.go:972
    summary: TestPrepare_SubagentTerms pins the wording of the subagent passage through wantPassage and wantAbsent substrings, which is static prose the plan says no test pins.
    fix: In TestPrepare_SubagentTerms keep the assertions on the effort setting, the effort value inside the passage and the skill names, and drop wantAbsent and the substring rows that assert surrounding wording such as the phrase Your skills are.
  - id: F8
    severity: LOW
    class: consistency
    origin: handler
    location: internal/shuttleengine/codexengine/command_test.go:13
    summary: Three tests the redundancy report lists as contained pin what their covering tests cannot see and carry no testtiming keep marker - TestBuildLaunchCmd, TestLoadConfig_Template and TestPrepare_ExpandsHomeInWritableRoots.
    fix: Add a testtiming keep line directly above each func Test line. TestBuildLaunchCmd - pins the literal flag order and shell quoting of the launch line, which TestPrepare_Launch and TestPrepare_SubagentTerms miss because they compare against buildLaunchCmd itself. TestLoadConfig_Template - pins the template's whole resolved key set, including the new defaults, which TestPrepare_InstallsGuardsFromItsInputs does not assert. TestPrepare_ExpandsHomeInWritableRoots - pins the leading tilde expansion in the launch line, the only test that sets a tilde writable root.
  - id: F9
    severity: NIT
    class: scope
    origin: lens:code-scope
    location: internal/heddlecli/verbs.go:223
    summary: The set help text drops the clauses listing the prime, beams and task pairs and the render_warnings case of a failure to list worktrees, beyond the agents-prompt addition card 2 asks for.
    fix: Restore the dropped wording in the lym heddle set help text, keeping only the agents-prompt additions, unless the reconcile render makes the removed clauses untrue, in which case leave it.
---

### [MEDIUM] Prepare resolves RepoRoot but not WorkDir for the user setup

**Location:** internal/shuttleengine/codexengine/codexengine.go:123
**Issue:** `readCheckout` returns a symlink-resolved root, and `Prepare` resolves `workDir` for the instruction-file cap, but the user-setup request takes the raw `c.dirs.Anchor`. Under a symlinked anchor, the skill listing's upward walk never meets `RepoRoot`, climbs to `/` and lists the `.claude/skills` of every ancestor including the home directory's. The `projectPath` match of project-scope plugin installs, the hook `cwd` and `CLAUDE_PROJECT_DIR` then use the unresolved form. The fixtures resolve their temp dirs, so no test sees it.

### [MEDIUM] docs/overview.md does not carry the codex launch behaviour this diff adds

**Location:** docs/overview.md:709
**Issue:** The paragraph describing the codex engine lists sandbox mode, writable roots, skill roots, guards and creel delivery in detail. It names none of the developer-instructions carrier and its argument cap, the subagent effort and thread cap, the Claude user setup, `project_doc_max_bytes` with the two-file override, or the pointers. The repository rule that a change's docs land in the same diff is not met.

### [MEDIUM] The unresolvable-config-directory behaviour is untested

**Location:** internal/shuttleengine/claudeengine/usersetup.go:538
**Issue:** The parent's Added-after-Discussion entry fixes what `UserSetup` does when `claudeConfigDir` fails. Every claudeengine fixture sets `source.claudeDir`, so the degrade path, its one warning and the kept project skills and above-root files have no row that would fail if broken.

### [LOW] Heddle-rendered override beside a CLAUDE.md with imports is named as unread

**Location:** internal/shuttleengine/codexengine/instructionfiles.go:594
**Issue:** The marker exemption sits inside the AGENTS.md branch. With a marked `AGENTS.override.md`, no `AGENTS.md` and a CLAUDE.md that has imports at the working directory, the import branch names the file although the override already carries it expanded, so the session is told to read it twice.

### [LOW] The unread-CLAUDE.md scan walks the whole checkout

**Location:** internal/shuttleengine/codexengine/instructionfiles.go:565
**Issue:** The walk skips only `.git`. On a tree with dependency or build directories, every launch pays the walk and may list vendored CLAUDE.md files, which also grows the developer instructions toward the argument cap.

### [LOW] Added failure paths and wirings with no covering row

**Location:** internal/shuttleengines/shuttleengines.go:82
**Issue:** Dropping `RunHook: claudeengine.RunPluginHook` from the registry leaves every test green. No test shows `lym heddle render` passes the worktree root that `recordImports` needs. The instruction-file stat refusal, a plugin with no install path, an unreadable plugins file, an invalid matcher and a non-command hook type have no row.

### [LOW] TestPrepare_SubagentTerms asserts the passage's wording

**Location:** internal/shuttleengine/codexengine/prepare_test.go:972
**Issue:** The `wantPassage` and `wantAbsent` fields assert substrings of a prose literal. The effort value and the skill names are behaviour; the surrounding wording is prose that the plan's constraint says no test pins.

### [LOW] Contained tests that pin what coverage cannot see carry no keep marker

**Location:** internal/shuttleengine/codexengine/command_test.go:13
**Issue:** The redundancy report lists `TestBuildLaunchCmd`, `TestLoadConfig_Template` and `TestPrepare_ExpandsHomeInWritableRoots` as contained. Their covering tests (`TestPrepare_Launch`, `TestPrepare_SubagentTerms`, `TestPrepare_InstallsGuardsFromItsInputs`) build the expected line with `buildLaunchCmd` itself or never set a tilde root or the template's other keys, so each test pins something they cannot see and must be marked, not folded.

### [NIT] The set help text loses clauses no decision asks to drop

**Location:** internal/heddlecli/verbs.go:223
**Issue:** The reword for `agents-prompt` also removes the list of rendered worktree kinds and the failed-listing case of `render_warnings`. Card 2 asks only for the `agents-prompt` additions.

## Rejected

- Order of the developer instructions against the record's parenthetical (subagent passage first): the record says the order is that of Claude's appended prompt file, which `claudeengine.go:260-266` writes as the creel head, then the subagent passage, then the passages, and the code matches it. Not a defect.
- D7 question: `lym-batching` and `lym-plan-edits-before-gate` already sit on `agents-file` in `shipped.json` since `22b74f24`, before the range, and their text names no Claude tool. `agents-prompt` renders into both prompt targets, `Show` refuses it with the way forward, and the reconcile re-renders every walked worktree; each is pinned by a test. Not a defect.
- lens:code-scope S1 (the import-based unread rule has no source in D9): card 10 states it in its Intent, so the plan licenses it.
- lens:code-correctness note on an empty `installPath` of a user-scope install hiding a later project-scope install: a plugin entry with no install path is malformed and already warned about; the case is too marginal to ask for a change.
- lens:code-correctness note on a fence closed by a line with an info string: Claude Code's own parser is not specified by the record, and the case needs two fences of equal length nested in one block; too marginal to ask for a change.
- lens:code-tests note that the empty-set omission of `developerInstructions` has no direct test: `Prepare` always carries the creel head, so the branch cannot be reached through the public surface; `TestBuildLaunchCmd` pins the omitted-key line.
- lens:code-tests note that no test builds the subagent passage together with standing passages: the order is fixed by one `append` sequence in `Prepare` and is Claude's own order; a row for it would pin the same sequence twice.
