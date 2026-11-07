```yaml
lens: code-affected-set
outcome: reported
```

**Task:** code-affected-set lens. For each change I looked for what it forces outside the touched lines and whether the diff covers it.

**Per change: set found, covered or not**

- **Go skill lists and `AppendSkills` removed.**
  - No reference to `AppendSkills`, `burlerSkills`, `darnSkills`, `fixerSkills`, `conflictSkills` or `roleSkills` is left anywhere outside `.git`, `_lym` and `.lym` (grep over the whole tree). Covered.
- **`Spec.ConfigRole` and `Seat.ConfigRole`.**
  - `ConfigRole:` appears at every spawn site in non-test code.
  - Each role in `agentconfig.Roles` has a site: arbiter, board-tidy, both bouncers, burler, conflict-resolver, darn, describer, discussion and its advisor, both drift seats, driver, fix-review, friction, gate-fixer, merge-fixer, orch, plan, pr-reviewer, raddle, sampler, tackler, both treadle roles and webster.
  - The verify fixer in `websterengine/verifyfixer.go` and the `gatecli` and `loomcli` fixers go through `gatefixer.SpawnOrAttach`, which sets `RoleGateFixer`. Covered.
  - Two sites deliberately carry no config role: `shuttlecli/open.go` (opened sessions) and `shedrecipe/entries_multillm.go` (the summary says recipe seats keep their recipe skills).
- **Skill resolution in `shuttleengine/run.go:565`.**
  - Later readers use the resolved `run.spec.Skills` (`run.go:807` and `run.go:818`), so the launch and the load turn see the merged list.
  - `open.go:84` reads `OpenSpec.Skills` only, which is correct since an opened session has no role. Covered.
- **Orch reload list.**
  - `orchcli/cli.go` resolves `Agents.Skills(RoleOrch, nil)` into `orchSkills`.
  - `watch.go` passes it to `NewWatcher`.
  - The start spec uses `ConfigRole: RoleOrch`.
  - Both paths resolve the same list. Covered.
- **New hub-wide module `agents`.**
  - It is registered in `configreg.Modules`.
  - `agentconfig` is added to the leaf-packages pattern file and to the degrading-callers list in the config-layers pattern file.
  - `docs/overview.md` has the new package entry (line 291) and the Claude engine paragraph (lines 677–680).
  - `docs/overview.md:348` (`languageconfig`) is still accurate.
  - The sibling `language` module is registered only in `configreg`, with no other module list to update. Covered.
- **Stencils.**
  - The five changed stencils each add one line that points at the system-prompt passage and restates no list.
  - No existing rule is dropped or weakened in the diff hunks.
  - All five already assume the Claude Agent tool, so the new pointer adds no new engine dependence. Covered.
- **Codex.**
  - `effortFor` and the passage are called only from `claudeengine`.
  - The codex path gets only the resolved skills. Covered.
- **Commit messages.**
  - No `Pattern-Departure:` trailer appears in the log, and none is needed.
  - The leaf-allowlist commit `b824252a` admits `agentconfig` to five engines' import allowlists. Covered.

**Findings**

1. **LOW, scope.** `internal/configcli/configcli.go:479`: the diff deletes the config help line "A value the loader refuses is refused, naming the key." The decision record does not ask for it, and the commit message "trim config help" gives no reason tied to the goal.
   - Evidence: the `configcli.go` hunk in the diff, and `configcli_test.go` changed one line alongside it.
   - Fix: restore the line, or justify the deletion.

No other member of the affected set was left wrong.

**Tool calls (4)**
1. A grep for stale skill-list names and `languageconfig`/`language.yaml` mentions; my context had only the diff, not the full tree.
2. A grep of `shuttleengine` for `.Skills` readers and of module-name lists; to check the resolved list reaches the launch, load and open paths.
3. A grep for stencil mentions of the subagent passage and the engine stencil directories; to check Claude-only wording and codex handling.
4. A grep of `docs/overview.md`, `configcli` and `hubreconcile` for config-module listings; to check no module list or seeding step was missed.
