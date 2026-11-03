---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: consistency
    location: docs/overview.md:423
    origin: lens:code-affected-set
    summary: The shuttle entry still says the codex engine has no skill loading or shell guards, which this diff adds.
    fix: "In docs/overview.md line 423, in the sentence that describes the registered codex engine (`it has no skill loading, shell guards, fork audit or creel hooks yet`), keep the no fork audit and no creel hooks claims and state instead that Codex loads skills as SKILL.md files the model reads by absolute path under codex.yaml's skill_roots, and installs every shell guard row of the shellguard table as a PreToolUse hook matching Bash with Codex's own steer and notice texts (apply_patch in place of Read, Edit and Write)."
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/shuttleengine/codexengine/codexengine.go:88
    origin: lens:code-tests
    summary: Prepare loads the hub gate config for the refuse list and refuses on a load failure, but no test drives either path.
    fix: "In internal/shuttleengine/codexengine/prepare_test.go add two rows to the existing Prepare tests: one writes a hub gate.yaml with a refuse_raw list and asserts the launch line carries a refuse-raw hook entry and the prompt file's deny notice names it (compare against installedGuards(list, inputs)); one writes a gate.yaml that exists but does not load and asserts Prepare returns an error beginning codexengine: and writes no prompt.md. Use the fixture's existing way of writing a hub layer."
  - id: F3
    severity: MEDIUM
    class: design
    location: internal/shuttleengine/codexengine/guards_test.go:68
    origin: lens:code-tests
    summary: No test pins the literal deny output or the Bash matcher, so the guard tests re-derive both from the code under test and a changed shape disables every guard unseen.
    fix: "In internal/shuttleengine/codexengine/guards_test.go add one assertion that denyDecision(\"x\") equals the literal {\"hookSpecificOutput\":{\"hookEventName\":\"PreToolUse\",\"permissionDecision\":\"deny\",\"permissionDecisionReason\":\"x\"}} and that every entry of guardEntries starts with the literal {matcher=\"Bash\",hooks=[. Keep the rest of TestGuardEntries_RenderEveryRowWithItsCodexText and drop its expectations that rebuild the entry from hookEntry(toolMatcherBash, guardCommand(row)), which restate the implementation."
  - id: F4
    severity: LOW
    class: design
    location: internal/shuttleengine/codexengine/codexengine.go:92
    origin: lens:code-tests
    summary: No Prepare test tells cfg.DenyPython and spec.DenyPublish apart, so swapping them in the guardInputs literal passes every test.
    fix: "In internal/shuttleengine/codexengine/prepare_test.go give TestPrepare_Launch's table rows, or a new row of TestPrepare_RefusesBeforeWriting's sibling table, one case with only Config.DenyPython true and one with only Spec.DenyPublish true, each comparing the launch line and prompt file against installedGuards with the matching single guardInputs field."
  - id: F5
    severity: LOW
    class: design
    location: internal/shuttleengine/codexengine/skillload_test.go:85
    origin: lens:code-tests
    summary: The skill load tests leave the custom_tool_call and local_shell_call payloads, a repeated load message, a truncated rollout line, ~/ root expansion and a bad codex.yaml uncovered.
    fix: "In skillload_test.go add rows to TestClassifySkillLoad for a custom_tool_call and a local_shell_call naming a skill tail (each loaded), a second equal load message after a read (the earlier read no longer counts), and a truncated JSON line between valid lines (skipped); add rows to TestSkillPath for a pre-release and a non-numeric version directory asserting the order F6 settles on; and extend skillEngine's callers with one engine whose skill_roots entry starts with ~/ under a t.Setenv HOME and one whose codex.yaml does not load (no skill resolves, one Warn). A test using t.Setenv is not parallel and says so in a comment."
  - id: F6
    severity: LOW
    class: design
    location: internal/shuttleengine/codexengine/skillload.go:140
    origin: lens:code-correctness
    summary: compareVersions is not a total order for mixed numeric and text fields, so the newest version can depend on glob order and a pre-release can beat its release.
    fix: "In compareVersions order a field pair by one total rule: when both fields are numbers compare them as numbers; when exactly one is a number the numeric field orders higher than the text one; otherwise compare as text. Treat a field with a numeric prefix and a suffix as text, and state the rule in the function's doc comment. Add the matching rows to TestSkillPath."
  - id: F7
    severity: LOW
    class: design
    location: internal/shuttleengine/codexengine/skillload.go:122
    origin: lens:code-correctness
    summary: filepath.Glob returns ErrBadPattern for a skill root holding a glob metacharacter, which skillPath reads as no root holds the skill, silently.
    fix: "In skillPath escape the root, the plugin and the name for filepath.Glob (a backslash before each of * ? [ and the backslash itself) so a literal path with a metacharacter resolves, and drop the err != nil branch's silent continue by treating the escaped pattern's ErrBadPattern as impossible."
  - id: F8
    severity: LOW
    class: design
    location: internal/shuttleengine/codexengine/skillload.go:53
    origin: lens:code-correctness
    summary: ClassifySkillLoad rebuilds the expected load line from the skill files present at classification time, so a plugin cache that gained a version since launch makes the line differ and the report degrade to unverified.
    fix: "In skillload.go make loadTurnCalls match a user message that equals the load line, or that starts with the fixed opening of skillLoadLine (hoist that opening into a constant used by skillLoadLine and the match), so the paths in the sent line need not be recomputed; keep the unverified result for a rollout with no such message. Add a row to TestClassifySkillLoad where a newer version directory appears after the line was built."
  - id: F9
    severity: NIT
    class: design
    location: internal/shuttleengine/codexengine/codexengine.go:124
    origin: handler
    summary: Prepare loads codex.yaml through loadConfigOrRefuse and SkillLoadMessage then loads it a second time, so a second load that fails launches with a Warn and no skill paths after the first load passed.
    fix: "In skillload.go split SkillLoadMessage into a method over c.skillRoots() and an unexported function that takes the already expanded roots; in Prepare call that function with the roots expanded from the codex Config it already holds, so one load serves the launch."
---

### [MEDIUM] Stale statement that Codex has no skill loading or shell guards

**Location:** docs/overview.md:423
**Issue:** The shuttle entry says the `codex` engine "has no skill loading, shell guards, fork audit or creel hooks yet".
This diff adds the first two, and `internal/shuttleengine/codexengine/doc.go` was updated for them, so the repository's own overview now contradicts the code.

### [MEDIUM] Gate config load in Prepare is untested

**Location:** internal/shuttleengine/codexengine/codexengine.go:88
**Issue:** `Prepare` now loads the hub's gate config so its `refuse_raw` list reaches the refuse-raw row, and refuses the launch when a present config fails to load.
No codexengine test writes a gate config, and `TestPrepare_Launch` passes a nil list, so neither the list reaching the hook and notice nor the refusal the change description claims is pinned.

### [MEDIUM] Deny output shape and Bash matcher are not pinned by a literal

**Location:** internal/shuttleengine/codexengine/guards_test.go:68
**Issue:** The guard tests and the run test build their expected deny output and entry from `denyDecision`, `hookEntry` and `toolMatcherBash`, the code under test.
A changed JSON key or matcher name would pass every test while Codex stopped honouring the guards, against the acceptance criterion that the hooks deny a matching command with the tested output.

### [LOW] No test separates the python and publish switches in Prepare

**Location:** internal/shuttleengine/codexengine/codexengine.go:92
**Issue:** `TestPrepare_Launch` turns both switches on and the standing-passages test turns both off, so exchanging `cfg.DenyPython` and `spec.DenyPublish` passes both.

### [LOW] Skill load branches left uncovered

**Location:** internal/shuttleengine/codexengine/skillload_test.go:85
**Issue:** Only the `function_call` payload type, one user-message reset, absolute roots and numeric versions are exercised.
The `custom_tool_call` and `local_shell_call` payloads, a repeated load message, a truncated line, `~/` root expansion, a `codex.yaml` that does not load and a non-numeric version directory have no row, though the focus names them as failure paths.

### [LOW] Version comparison is not a total order

**Location:** internal/shuttleengine/codexengine/skillload.go:140
**Issue:** Numeric fields compare as numbers and the rest as text, so `2`, `10` and `1a` form a cycle and the newest version then depends on glob order.
A field `0-beta` text-compares above `0`, so a pre-release beats its release, and a hash-named directory orders arbitrarily against `1.1.0`.

### [LOW] A glob metacharacter in a skill root silently hides every skill

**Location:** internal/shuttleengine/codexengine/skillload.go:122
**Issue:** A root holding `[` makes `filepath.Glob` return `ErrBadPattern`, which `skillPath` treats like an empty match, so each skill is reported unknown with no Warn.

### [LOW] Classification depends on the plugin cache as it is at turn end

**Location:** internal/shuttleengine/codexengine/skillload.go:53
**Issue:** `ClassifySkillLoad` recomputes the load line from the files present when it runs.
A newer plugin version that lands between launch and turn end changes the line, so no user message in the rollout equals it and verification and the retry are skipped.

### [NIT] codex.yaml is loaded twice per launch with a skill list

**Location:** internal/shuttleengine/codexengine/codexengine.go:124
**Issue:** `Prepare` holds the loaded `Config` but `SkillLoadMessage` reloads codex.yaml through `skillRoots`, so a failure of only the second load launches with a Warn and a line that names no skill file.

## Rejected

- Lens code-correctness finding 1, the tail substring marks another plugin's same-named skill loaded: the decision record's D9 prescribes the `skills/<name>/SKILL.md` tail as the match, so the false-loaded case is inside the decision and a finding about it is about the answer key.
- Lens code-tests finding 1 as a separate LOW: merged into F3, which carries the same missing literal and raises it to MEDIUM against the acceptance criterion.
- Lens code-tests finding numbering 2 and 3 appear as F4 and F2: kept, with F2 stated against the gate config path and F4 downgraded to LOW since each switch is covered in `guards_test.go` and only Prepare's mapping is unpinned.

## Focus verdicts

- Per-engine steer and notice (D10): `SteerFor` and `NoticeFor` fall back to the plain text, `internal/shellguard` imports only the standard library besides its two admitted leaves, and the diff leaves every Claude render byte-identical (the recomposed constants join to the same strings). Not a defect.
- `guards.go` (D10): every row is rendered in table order as a Bash `PreToolUse` entry and the deny notice joins every installed row's Codex notice. Not a defect; F3 concerns pinning only.
- Guards test for every row (D10): `TestInstalledGuards_CoverEveryRow` derives its expectation from `shellguard.Rows`, as the Claude test does; see F2 to F4 for what stays unpinned.
- `skillload.go` (D9): the load line asks for each `SKILL.md` by absolute path in one turn, resolves to the newest version under the roots, leaves an unknown skill out, and classifies from the rollout tail with unverified on an unreadable rollout. Defects only at the edges in F5 to F8.
- `codexengine.go` and `command.go` (D6): the prompt file opens with the deny notice and the launch carries the hooks and first turn; nothing widens into D7 or D8.
- Scope: the diff stays within D9 and D10, and its change description matches it.
- Test economy: no duplicated test found; the gaps are the ones above.
