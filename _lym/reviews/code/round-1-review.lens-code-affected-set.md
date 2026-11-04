```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens for the board-agent-button diff (`925431bd..HEAD`).

**List: per change, the set found**

1. `buildOpenCmd` signature gained `firstTurn`.
   - Set: the one call site in `OpenLaunch`, two calls (`command.go:459-460`). A grep for other callers outside `command.go` found none. Covered.
2. `OpenSpec.Skills` and `--skill` (`engine.go`, `shuttlecli/open.go`, `Runner.Open`, `OpenLaunch`).
   - Set: the CLI flag and its help text, the `Runner.Open` refusal, the `OpenSpec` and `SessionOpener` doc comments, the package docs `shuttleengine/doc.go` and `claudeengine/doc.go`, and `docs/overview.md`. All were updated in the diff. Covered.
   - Codex: `codexengine` has no `OpenLaunch`, so no engine ignores `spec.Skills` today. The summary's open risk covers the peer task. Nothing to fix now.
   - `reedcli/add.go:64` mentions `lym shuttle open --role` only in passing, with no flag list. Unaffected.
3. `orchSkills` gained `ly:board-policy`.
   - Set: `start.go:196`, the watcher's reload list at `watch.go:126`, `start_test.go`, and the `orchcli` doc. The first four are covered.
   - The other `ly:board` literals are not copies of the orch skill list. `dobbyengine/tidy.go:30` is the tidy agent's own single skill. The `orchengine` tests use their own fixtures (`reloadSkills` at `watcher_test.go:1164`). Not affected.
4. Orch role trim: the rules for priority, `recipe: darn` and the policy bullets moved out of `orch-role.md`.
   - Set: the only Go consumer of the role file is `orchengine/prompt.go:53`, which fills the two markers. Both markers are still in the section (`{{.decision_note_title}}` is kept). No test or doc asserts the removed phrases; a grep found them only in `board-policy/SKILL.md`. Covered.
   - Every launch path loads the skill: the `Start` spec and the watcher reload both use `orchSkills`.
5. Prime `board` button in `template_posix.yaml` and `template_windows.yaml`.
   - Set: both templates, and `config_test.go`, which asserts both against one `wantShipped`. Covered.
   - The reed `doc.go` describes only the button mechanism, with no group contents. `docs/` outside `overview.md` has no button text. Nothing stale.
6. PATTERN entries.
   - The diff declares no PATTERN change, so there is nothing to land.
   - `PATTERN-parent-directive` (skills go on the launch spec) holds for the button, since the skills ride the `lym shuttle open` launch spec.
   - `PATTERN-stencil-ownership` and `PATTERN-lym-instructions-in-code` are met by a shipped plugin skill, as `ly:board` already is.
   - No `Pattern-Departure` trailer is present, and none is needed.
7. Commit messages.
   - Commit 1 is `585899c2`; commit 2 is `c6404839`. Both match their diffs.
   - Commit 2 says the orch "now loads" the skill at launch, which `start.go:36` bears out.

**Findings**

1. **LOW · consistency · `plugins/ly/skills/board-policy/SKILL.md:57` and `INDEX.md:6`**
   - The skill's `description` and the INDEX row both give the policy as "discuss before writing", but the body has no such rule. It reaches the body only as "Talk each case through ... before writing it", in the button-session section at line 57.
   - A session that follows the policy, the orch included, never meets "discuss before writing" as a rule. The orch role's old board section did not state it either, so no rule is lost.
   - Evidence: `board-policy/SKILL.md:3`, `board-policy/SKILL.md:54-61`.
   - Fix: add a Notes bullet, or drop the phrase from the description and INDEX row.

2. **LOW · scope · the orch in the middle of a run when the binary changes**
   - A live orch session started before this binary has the trimmed role file but never loaded `ly:board-policy`, because skills load at launch and reload only on a cycle.
   - It therefore lacks the priority and darn-recipe rules until its next clear cycle or restart.
   - Evidence: `orchcli/start.go:36`, `contracts/orchrole/orch-role.md:206`.
   - Fix: tell the operator to refresh the orch after landing; no code change.

No other member of the affected set is left wrong.

**Tool calls**
1. A grep for `buildOpenCmd(` callers, the removed rule phrases, `lym shuttle open` in docs and code, and `buttons` in `docs/`. My context had the diff but not the whole-repo caller sets.
2. A grep of `ly:board` in `dobbyengine` and `orchengine`, to tell whether the literals were copies of the orch skill list. They are not.
