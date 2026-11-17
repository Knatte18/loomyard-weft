---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lym/plan/08-plugin-session-start-contexts.md:57
    summary: TestRunPluginHook is planned at tier0 but runs real sh -c processes through RunPluginHook, which PATTERN-test-speed bars from tier0
    fix: In card 8's Intent, replace the single heading "Tests, tier0:" with two tier statements, keeping TestUserSetup_PluginSessionStartContexts at tier0 (it uses a fake RunHook) and stating on its own line that TestRunPluginHook is tier1, since its rows spawn sh and sleep; the test then sits in a test file whose functions all name tier1, apart from the tier0 tests.
    origin: lens:repo-invariants
  - id: F2
    severity: MEDIUM
    class: design
    location: _lym/plan/05-codex-developer-instructions.md:47
    summary: The 128 KiB refusal lets a setting of exactly maxArgumentBytes go ahead, but execve counts the terminating NUL, so a 131072-byte argument fails with E2BIG
    fix: In card 5's Intent, make developerInstructions refuse when the setting's length is at least maxArgumentBytes (the longest argument execve takes is maxArgumentBytes-1 bytes, MAX_ARG_STRLEN counting the NUL), replace "A setting of exactly maxArgumentBytes bytes goes ahead" with "A setting of maxArgumentBytes-1 bytes goes ahead", and change the test rows so AtTheArgumentCap sizes the setting at maxArgumentBytes-1 bytes and goes ahead while OverTheArgumentCap sizes it at exactly maxArgumentBytes and is refused.
    origin: lens:guard-reach
  - id: F3
    severity: MEDIUM
    class: scope
    location: _lym/plan/10-codex-instruction-files.md:15
    summary: Card 10 adds project_doc_max_bytes to every Prepare launch line but does not list TestPrepare_SubagentTerms, whose card-6 rows compare whole want lines, so card 10's gate fails on it
    fix: Add `plan:internal/shuttleengine/codexengine#TestPrepare_SubagentTerms` under card 10's **Edit:** after `internal/shuttleengine/codexengine#TestPrepare_InstallsGuardsFromItsInputs`, and extend the Intent's last test bullet to say that TestPrepare_SubagentTerms's want lines also add projectDocMaxBytes defaultProjectDocMaxBytes, its fixture holding no instruction file.
    origin: lens:affected-set
  - id: F4
    severity: LOW
    class: consistency
    location: _lym/plan/10-codex-instruction-files.md:47
    summary: unreadClaudeFiles skips the working directory's CLAUDE.md whenever AGENTS.override.md exists there, but a repository's own override without heddle's Marker is left untouched by Render and does not carry CLAUDE.md
    fix: In card 10's Intent, change the exception to apply only when the working directory's AGENTS.override.md opens with heddleengine's Marker (read the file's head and compare, as Render does), so an override heddle did not write leaves the CLAUDE.md named; add a TestPrepare_InstructionFiles row with an unmarked AGENTS.override.md at the anchor beside AGENTS.md and CLAUDE.md that expects the CLAUDE.md named.
    origin: lens:guard-reach
  - id: F5
    severity: LOW
    class: design
    location: _lym/plan/04-codex-file-carries-both.md:46
    summary: Render's empty-Root branch guards a state no production builder produces once requestOf and heddlecli's renderInto both set Root
    fix: In card 4's Intent, state that the empty-Root branch exists for request builders that carry no worktree root, namely the existing TestRender fixture's f.request(), and keep it as the documented no-record case; or drop the sentence and the NoRoot row and set Root in f.request() instead. Either way, name in the Intent which builder reaches the branch.
    origin: lens:guard-reach
  - id: F6
    severity: LOW
    class: consistency
    location: _lym/plan/07-claude-user-setup-source.md:52
    summary: A read error of any CLAUDE.md above the repository root refuses every Codex launch, though the walk reaches machine directories the user may not own and the way forward assumes they can fix the file
    fix: In card 7's Intent, keep the refusal for the user's global file only; make an unreadable CLAUDE.md above RepoRoot a Warnings entry naming the file, left out of InstructionFiles; change the TestUserSetup_InstructionFiles row list to add an unreadable above-root file giving a warning and no error.
    origin: lens:guard-reach
  - id: F7
    severity: LOW
    class: decision
    location: _lym/plan/07-claude-user-setup-source.md:47
    summary: Card 7 prescribes no outcome when claudeConfigDir fails to resolve the config directory in production
    fix: In card 7's Intent, state that a claudeConfigDir error makes UserSetup return the zero UserSetup with one Warnings entry naming the cause and CLAUDE_CONFIG_DIR or HOME as the way forward, never an error, as sessionScopeFor does with an unreadable plugins file.
    origin: lens:guard-reach
  - id: F8
    severity: LOW
    class: consistency
    location: _lym/plan/02-agents-prompt-target.md:7
    summary: The Target type's doc says RendersInto names the file targets of a shared file target, which agents-prompt makes false, and card 2 does not list Target
    fix: Add `internal/heddleengine#Target` under card 2's **Edit:** before `internal/heddleengine#targetTable`, and add an Intent sentence that its doc comment says RendersInto names the targets of either delivery a shared target's entries reach.
    origin: lens:affected-set
  - id: F9
    severity: LOW
    class: consistency
    location: _lym/plan/03-reconcile-renders-file-targets.md:14
    summary: Card 3 lists PATTERN-heddle-instructions.md under Extends while its Intent inserts the reconcile into the existing render-site sentence, which rewrites content already there
    fix: Move `pattern/PATTERN-heddle-instructions.md` from card 3's **Extends:** to its **Edit:** group, after `internal/fabricengine_test#TestWorktreeSeam_Events`.
    origin: handler
  - id: F10
    severity: LOW
    class: design
    location: _lym/plan/08-plugin-session-start-contexts.md:51
    summary: A hook run is placed in its own process group and registers no runproc record, so when the lym process running Prepare dies mid-run, the hook outlives it, with no timeout left and nothing that can end it
    fix: In card 8's Intent, state the bound that a hook run's process group is killed through cmd.Cancel only while the starting lym process lives, and register each run with runproc.ChildRecord under the run's owner for its lifetime, removing nothing itself, so a teardown's runproc End reaches a hook left by a killed Prepare; add the record to TestRunPluginHook's timeout row only if runproc offers a test seam, else name the bound as review-held.
    origin: handler
  - id: F11
    severity: LOW
    class: decision
    location: _lym/plan/08-plugin-session-start-contexts.md:36
    summary: The hook environment sets CLAUDE_PLUGIN_ROOT but not CLAUDE_PROJECT_DIR, which Claude Code sets for every hook, so it does not run hooks the way Claude Code runs them as D8 asks
    fix: In card 8's Intent, add CLAUDE_PROJECT_DIR set to req.RepoRoot to Env beside CLAUDE_PLUGIN_ROOT, and extend the TestUserSetup_PluginSessionStartContexts row that checks the recorded run to assert both variables.
    origin: handler
---

The plan carries every decision of D5 to D9 and the record's Goal, Delivers and Exports. It breaks one PATTERN rule, has one refusal boundary that is off by one byte, and one card misses a test its change breaks at its own gate.

### [BLOCKING] TestRunPluginHook spawns processes at tier0

**Location:** `_lym/plan/08-plugin-session-start-contexts.md:57`, rows at lines 67–69.

**Issue:** The card heads its tests "Tests, tier0" and plans `TestRunPluginHook` with rows that run `sh -c` commands through the real `RunPluginHook`, including `sleep 5; true` under a 100ms timeout.
`PATTERN-test-speed` defines `tier0` as "offline, no spawn" and puts subprocesses at `tier1` (`pattern/PATTERN-test-speed.md:3,12-13`).
The static `exec.Command` scan in `tierpurity_test.go` never sees the spawn, because the call sits in the production file. Nothing would catch it before every card gate starts spawning shells.

### [MEDIUM] The 128 KiB refusal's boundary admits an argument execve rejects

**Location:** `_lym/plan/05-codex-developer-instructions.md:47-49`, test row at line 67.

**Issue:** The card lets a setting of exactly `maxArgumentBytes` (131072) bytes go ahead, and pins that with the `AtTheArgumentCap` row.
Linux's `copy_strings` measures each argument with `strnlen_user`, which counts the terminating NUL, and refuses when the result exceeds `MAX_ARG_STRLEN`. So the longest argument `execve` accepts is 131071 bytes.
Reed stages the launch line in a script the pane sources (`internal/reedengine/spawn.go:112-119`), so the shell execs `codex` with the setting as one argv element.
A 131072-byte setting therefore fails with E2BIG, the failure D5's refusal exists to prevent, and the plan's test asserts that it goes ahead.

### [MEDIUM] Card 10 does not update card 6's full-line test

**Location:** `_lym/plan/10-codex-instruction-files.md:10-18`, the test note at line 66.

**Issue:** Card 6 builds `TestPrepare_SubagentTerms`'s want lines through `developerInstructions` and `tomlString` (`06-codex-subagent-terms.md:58-59`), so they are whole launch lines from `Prepare`.
Card 10 adds `-c project_doc_max_bytes=<n>` to every `Prepare` line and updates only `TestBuildLaunchCmd` and the four full-line Prepare tests.
Every row of `TestPrepare_SubagentTerms` then fails at card 10's tier0 card gate.
Card 9's `TestPrepare_UserSetup` asserts an order inside the developer instructions rather than a whole line, so it is not affected.

### [LOW] The unread-CLAUDE.md exception trusts any AGENTS.override.md

**Location:** `_lym/plan/10-codex-instruction-files.md:47`.

**Issue:** The exception assumes an `AGENTS.override.md` at the working directory is heddle's, which carries both files.
`Render` leaves an unmarked override untouched with a warning (`internal/heddleengine/render.go:149-151`). In that case Codex reads only the repository's own override, and the CLAUDE.md beside it is neither carried nor named.

### [LOW] Render's empty-Root branch has no production builder

**Location:** `_lym/plan/04-codex-file-carries-both.md:40,46,61`.

**Issue:** After card 4, `requestOf` (`internal/heddleengine/seam.go:80`) and `renderInto` both set `Root`.
The only builder that leaves it empty is the test fixture's `f.request()`. The card's rule for a branch upstream rules out asks the Intent to name what reaches it, and the Intent does not.

### [LOW] An above-root CLAUDE.md the user cannot read blocks every Codex launch

**Location:** `_lym/plan/07-claude-user-setup-source.md:49-52`.

**Issue:** The walk above `RepoRoot` reaches machine directories, as the card's own test note says (line 74).
An unreadable CLAUDE.md there makes `UserSetup` return its one error, so every Codex launch is refused. A user who does not own the file cannot take the way forward, which is to fix its permissions or remove it.

### [LOW] No outcome for a config directory that cannot be resolved

**Location:** `_lym/plan/07-claude-user-setup-source.md:47`.

**Issue:** In production `claudeDir` is empty and resolves through `claudeConfigDir`, which errors when no home directory resolves (`internal/shuttleengine/claudeengine/scope.go:140-148`).
The card leaves open whether that refuses the launch or only adds nothing.

### [LOW] The Target doc goes false

**Location:** `_lym/plan/02-agents-prompt-target.md:6-16`.

**Issue:** `internal/heddleengine/target.go:46-47` says `RendersInto` "names the file targets whose files carry the entries of a shared file target".
`agents-prompt` is a shared prompt target that renders into prompt targets, and card 2 does not list `Target` to correct its doc.

### [LOW] A PATTERN background rewrite listed as Extends

**Location:** `_lym/plan/03-reconcile-renders-file-targets.md:14`, Intent at line 49.

**Issue:** The Intent names the reconcile "beside fabric's seam, `lym heddle render` and `lym heddle set` and `remove`". That is the existing sentence at `pattern/PATTERN-heddle-instructions.md:22-24`, which the card rewrites rather than adds to.
`Extends` is for refs the card only adds to.
`lym loom validate-plan --overlay` with this move, F3's and F8's target changes applied together reports no blocking finding beyond the baseline.

### [LOW] A hook run left by a killed Prepare has no owner

**Location:** `_lym/plan/08-plugin-session-start-contexts.md:49-54`.

**Issue:** `setHookProcessGroup` starts each hook in its own process group, and the card registers no `runproc` record.
`cmd.Cancel` enforces the timeout only while the lym process running `Prepare` lives. If that process is killed, for example by a run teardown, the hook's group is not the killed group, and no record lets `runproc`'s `End` reach it.
The record's `PATTERN-child-processes` constraint asks the hook runs to be "registered as the pattern says". The Shared Decision reads this as no registration, and leaves this case's bound unstated.

### [LOW] The hook environment lacks CLAUDE_PROJECT_DIR

**Location:** `_lym/plan/08-plugin-session-start-contexts.md:36`.

**Issue:** D8 asks for each hook to be run "the way Claude Code runs it".
Claude Code sets `CLAUDE_PROJECT_DIR` for every hook, beside `CLAUDE_PLUGIN_ROOT` for a plugin's hooks. A hook that reads it gets an empty value at `Prepare` and behaves differently from the same hook under Claude.

## Rejected

- Developer-instructions order (D5): the record's parenthetical lists the subagent passage first, but its governing clause is "in that file's order". The file (`claudeengine.go:262-264`, `settings.go#appendPromptText`) holds the deny notice and creel sentence first, then the subagent passage, then the passages, which is the order the overview's Shared Decision gives. Not a defect.
- Card 3's `fabricengine.NotifyWorktreeMadeAt` as scope the record does not license: D7 licenses the reconcile render. `hubreconcile`'s allowlist (`internal/hubreconcile/seam_enforcement_test.go:14-25`) bars `lymcwd`, and `EventFor` needs a `lymcwd.Location`, so a path-taking fire in fabric is the means. A path-based notify is a general seam under `PATTERN-fabric-boundary`. Not a defect.
- repo-invariants' possible break of card 5's `TestPrepare_AppendsStandingPassages` under cards 6 and 10: card 5 has that table assert the developer instructions' content, not a whole line, and the lens marks it as unconfirmed. Not filed.
- guard-reach's nil `RunHook` path, which production never reaches: the board entry's Exports license it as a seam for launch-manifest-parity, as the lens itself concluded.
- Focus checks that found nothing:
  - Card order: no card uses a symbol a later card creates. Cards 6, 9 and 10 build on card 5's `developerInstructions`, cards 8 to 10 on card 7's seam, and card 10's test on card 9's `Seams`.
  - Verify lines: no card has one.
  - Research steps: no card carries one.
  - Member glyphs: the Edit cards with source targets list member glyphs. Re-signed members carry the arrow: `fileText`, `NewFromConfig` and `readyHome`.
  - Test economy: no planned test duplicates an existing one. `TestLoadConfig_RefusesInvalidValues` covers refusals no codexengine test reaches today (a grep of the tests for `is none of` and `MarkInvalid` finds none), and the subagent-passage rows assert Go-filled values, not static prose.
