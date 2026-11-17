```yaml
lens: forced-edits
outcome: reported
```

Forced-edits lens: what each card's change forces outside the members it names.

**Findings**

1. **MEDIUM. Card 5 (D5): stale prompt-file prose that no card fixes.**
   After card 5, `prompt.md` holds the spec's prompt alone, so several comments become false:
   - `internal/shuttleengine/codexengine/guards.go:2` says the deny notice "opens the Codex prompt file".
   - `internal/shuttleengine/codexengine/codexengine.go:177` (`internal/shuttleengine/codexengine#Codex.OpenLaunch`) gives the reason "no prompt file to hold the creel trust sentence". That sentence now rides `developer_instructions`.
   - `docs/overview.md:711` says a Codex run carries "the creel trust sentence in its prompt file".

   No card lists `guards.go`, `Codex.OpenLaunch` or `docs/overview.md`.

2. **MEDIUM. Cards 5, 6, 9 and 10: `doc.go` lines are rewritten under `**Extends:**`.**
   Card 5 rewrites `internal/shuttleengine/codexengine/doc.go:63`, `:86` and `:126`, which describe what the prompt file holds. That changes existing text, so the ref belongs under `**Edit:**` (rubric "Extends on a ref the card rewrites").
   The same holds for card 4: `internal/heddleengine/doc.go:13` says the codex-file target opens with "AGENTS.md, else its CLAUDE.md", and card 4 rewrites that line.
   It also holds for card 2 if it changes the existing prompt-target sentence at `internal/heddleengine/doc.go:48-49`.

3. **MEDIUM. Cards 5, 6 and 10: `TestPrepare_SkillsLoadFirst` is missing from the Prepare test sets.**
   `internal/shuttleengine/codexengine#TestPrepare_SkillsLoadFirst` (`skillload_test.go:289-310`) asserts that the launch line holds `"--dangerously-bypass-hook-trust '"+loadLine+"'; "` and that `prompt.md` is the pointer target.
   It passes only if the new `-c` settings sit before `--dangerously-bypass-hook-trust`. The cards fix that order only for the developer instructions ("after the effort setting") and the agents keys. Card 10 puts `project_doc_max_bytes` "beside the fallback-file setting".
   This is likely to keep passing, but no card lists the test, and cards 5, 6 and 10 each say "the four full-line Prepare tests" without naming it.
   Also missed by that set:
   - `TestPrepare_ConcurrentLaunchesLoseNoRoot`, `TestPrepare_Warnings` and `TestPrepare_RefusesBeforeWriting` go through Prepare. Card 9's new source call and card 10's walk run in them unlisted, though they assert no full line.

4. **MEDIUM. Card 7: the `writeInstalledPlugins` helper cannot serve the new tests.**
   `internal/shuttleengine/claudeengine#writeInstalledPlugins` (`scope_test.go:36-50`) writes every install as `{"scope":"user","installPath":"/plugins/"+id}`. That is a fixed path outside the temp directory, with no `projectPath`.
   `TestUserSetup_SkillListing` needs plugin directories that hold `skills/*/SKILL.md`, plus a project-scope install for another repository.
   Card 7 lists the helper under `**Uses:**`, so it may not change it, and no new helper is declared. The helper must move under `**Edit:**` with a new parameter, or a new helper must be declared as a `**Create:**` handle.

5. **LOW. Card 9: the shuttleengines import allowlist is not checked.**
   The registry now calls `claudeengine.NewUserSetupSource` and `claudeengine.RunPluginHook`. Both paths are already allowed in `internal/shuttleengines/seam_enforcement_test.go:15-22`, so nothing breaks.
   But card 9's `TestNew_CodexLaunchCarriesTheClaudeUserSetup` sets `CLAUDE_CONFIG_DIR`. `UserSetupSource` must read that variable through `claudeConfigDir` when its `claudeDir` is empty. Card 7 states this, so there is no defect here.

6. **LOW. Card 9 / constraint `PATTERN-shuttle-provider-seam`: the background file is not updated.**
   `pattern/PATTERN-shuttle-provider-seam.md` lists each route by which an engine's capabilities leave the engine through `internal/shuttleengines` (`EnvScrub`, `SessionNamers`, `RecordReaders`). The new route, claudeengine's user-setup source reaching codexengine through a `shuttleengine` interface wired in the registry, is the decision record's own constraint text.
   No card extends that background file.

7. **LOW. Card 2: wording.**
   The `Show` refusal text changes at `internal/heddleengine/render.go:341`. Its doc comment at `render.go:334` ("renders into the files of other targets") is inside `Renderer.Show`, which card 2 lists, so it is covered.
   The comment at `internal/heddleengine/target.go:36` covers only `agentsFileTarget` and stays true, so card 2 has no defect beyond finding 2.

Tool calls:
- One grep across the tree for callers of `NewFromConfig`, `promptText` and `fileText`, the Show refusal text, the subtest names, the hubreconcile helpers and template copies. My context lacked callers outside the files I had read.
- One grep of the codexengine test files for launch-line builders and `prompt.md` reads, to find Prepare tests outside the read part of `prepare_test.go`.
- One read of `TestPrepare_SkillsLoadFirst`, the `writeInstalledPlugins` helper, the prompt-target test uses and prompt-file comments, to confirm what those tests and helpers assert.
- One grep of `docs/overview.md` and the shuttleengines import scan, because the docs and the allowlist were not in my context.
- One read of the shuttleengines and hubreconcile import allowlists, to check the new imports against them.
- One grep for import allowlists in heddleengine, shuttleengine, fabricengine and heddlecli, to check card 4's new imports. It found none.
