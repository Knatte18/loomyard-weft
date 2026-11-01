```yaml
lens: code-affected-set
outcome: reported
```

**Lens code-affected-set.** I traced what the diff's changes force outside the lines they touch. The list below has one entry per change, and two findings follow it.

1. **`Passages` and `StandingPassages` gain a `role` parameter.**
   - The set is `internal/shuttleengine/passages.go` (`PassageReader`, `StandingPassages`), `claudeengine.go:233` (the one production caller), `cmd/lym/main.go:161` (`InstallPassageReader(Renderer{}.Passages)`), `render_test.go` and `prepare_test.go`.
   - The shuttleengine `doc.go` and the `main.go` comment are updated. Covered.
   - `claudeengine.go:233` passes `spec.Role`. `orchcli/start.go:191` sets `Role` to `agentname.RoleOrch`, so the orch launch carries a role. Covered.
   - Codex has no production `StandingPassages` caller and appends no passages. The heddleengine `doc.go` says so ("an engine whose launch appends no passages delivers neither"). Covered.

2. **Target table: new `Engine` and `Role` fields, the `orch-prompt` row, `promptTargetsFor`.**
   - The set is `target.go`, `render.go` (`Passages`, `passagesOf`, `Show`), `heddlecli/cli.go` Long help, `heddlecli/verbs.go:120` and the heddleengine `doc.go`.
   - `TargetNames` is derived from the table, so help and validation pick up the new target automatically. No stale list of targets was found. Covered.

3. **`heddlecli.ChangeStore` is added.**
   - The set is `cli.go`, the heddlecli `doc.go`, `PATTERN.md` and `pattern/PATTERN-heddle-instructions.md`.
   - The declared PATTERN change (writers may include "their write path") lands in both `PATTERN.md` and the pattern file, with matching text. Covered.

4. **The hub-wide and per-orch rules files are removed.**
   - The set is `orchengine.Paths`, `orchroute/paths.go`, `orchcli/paths.go`, `RenderRoleFile` (three callers: `orchcli/start.go:580`, `watcher.go:193`, `resumecontext.go:27`), the `orch-template-role.md` markers, `markers_test.go`, and the `orchengine` and `orchcli` doc.go files.
   - A repo-wide grep finds no dangling reference to `HubRulesPath`, `OwnRulesPath`, `RulesRel`, `readRules` or the old markers. The role-stencil header comment is updated. Covered.

5. **The `rules` verb envelope changes from `{path, committed, unpushed}` to `{slug, cleared, warning}`.**
   - The set is `rules.go`, the orch role stencil text, the orchcli `doc.go` and the tests.
   - The grep found no other reader of the old keys. Covered.

6. **Existing hubs that already hold a Bolt `_lym/orch/rules.md`.** This is not covered; see finding 1.

**Finding 1 — MEDIUM, class `scope`.** The old rules files are no longer read, and nothing handles or mentions existing content.
- Location: `internal/orchengine/prompt.go` (the `readRules` removal), `internal/orchcli/rules.go`, `contracts/stencils/orch/orch-template-role.md:266-272`.
- A hub with a committed Bolt `_lym/orch/rules.md` or a per-orch `_lym/orch/rules.md` loses those rules from every orch with no signal.
- The verb Long help and the role text say a draft "starts as a copy of the current text" from `lym heddle get orch-rules`. That entry does not exist on such a hub, so the old content is not offered as a starting point.
- The Idea says only that the files "go away". It does not say whether existing content should be migrated or reported, and the commit message and docs are silent too.
- Evidence: `git diff` of `prompt.go` and `rules.go`, and the grep above with no migration or notice path.

**Finding 2 — LOW, class `consistency`.** A fixture comment is now misleading.
- Location: `internal/configcli/configcli_integration_test.go:441-443,497`.
- The test still writes `_lym/orch/rules.md` as the "unrelated file". The path is a leftover of the removed feature.
- The comment edit is harmless, but the fixture now names a path nothing reads.

No finding on: the stencil wording (the "Orch rules" section keeps the "add to this role, override none" and "contradiction goes to operator" rules); PATTERN entry text versus the declared change; or commit-message claims for the affected set. I raised no `Pattern-Departure:` question because the commits carry no trailer.

Tool calls:
1. A grep across `.go`, `.md` and other files for old rules identifiers, `RenderRoleFile` callers and `*-prompt` references. My context lacked a repo-wide dangling-reference search.
2. `git diff` of the orchengine, orchcli and configcli docs, watcher, resumecontext, state and markers files, plus `Role:` assignments. Those hunks were not in the manifest.
3. A grep for other `claude-prompt` mentions and `spec.Role` or `Role:` sites in shuttle, to confirm no other launch path builds a spec without a role.
