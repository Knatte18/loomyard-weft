```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. For each behaviour the diff adds or changes, I looked for the test row that would fail if it broke.

**Per behaviour: pinning test, or none**

1. `lym shuttle open --skill` flag, repeatable and in order.
   - Pinned by `TestOpenCmd_ModelAliasEngine`, row "skills are passed in the order given" (`internal/shuttlecli/cli_test.go:762`).
   - That row asserts `Skills` equals `[ly:board-policy, ly:board]`. The other rows assert `Skills` is empty.

2. `Runner.Open` refuses skills for an engine without `SkillLoader`, before the marker directory or a strand exists.
   - Pinned by `open_test.go:54`. The default `openerFakeEngine` has no `SkillLoader`.
   - The loop at `open_test.go:82-88` asserts no `AddStrand` call and an absent marker directory.
   - Weak spot: `fakeEngine`'s `OpenLaunch` is never reached in that row. Moving the check after `OpenLaunch` would still pass the marker-directory assertion, because the fake writes nothing.

3. Skills accepted for an engine that does implement `SkillLoader`.
   - No `Runner.Open` row passes skills through to `OpenLaunch` for a `SkillLoader` engine.
   - The `shuttlecli` row uses the `shuttlefake` engine, which implements `SkillLoader`. It reaches `OpenLaunch` only through a recorder, so it covers the CLI-to-spec path.
   - If the `Runner.Open` guard were inverted, only the refusal row at `open_test.go:54` would fail. That row covers the inversion, so no finding.

4. `skills` added to the "opened session" log line (`open.go:112`).
   - No assertion. The log capture test checks only `permissionMode=bypass` (`open_test.go:118`).
   - Observability only.

5. Claude `OpenLaunch` puts the skill-load message on the fresh line as the first turn, before the session flags, as a quoted positional argument.
   - Pinned by `TestOpenLaunch`, row "skills ride the fresh line..." (`command_test.go:447-457`).
   - It asserts the exact prefix `Invoke + quoted message + " --session-id "`.

6. The resume line carries no message, and `OrElse(resume, fresh)` is the fallback.
   - Pinned by the same row (`command_test.go:453-456`).
   - It builds the expected resume line by stripping the message from `Cmd` and swapping the flag, then compares to `ResumeCmd` exactly.

7. Over-limit skill-load message is refused before anything is written.
   - Row at `command_test.go:459`, with `board: true`, `wantErr "skill-load message is"`.
   - Whether this row asserts the directory stays uncreated depends on the shared checks at `command_test.go:509-523`. I did not read that code, so I can't say.
   - A different case is untested: a message exactly at the limit. That is a boundary row, and `>` versus `>=` would go unnoticed. I am not raising it, because it matches the existing run-path check, which has no such row in this diff.

8. No skills means an empty first turn, with the line unchanged.
   - Pinned by the existing `TestOpenLaunch` rows, which assert the exact line.
   - `buildOpenCmd` with an empty `firstTurn` is covered implicitly by every pre-existing row.

9. `orchSkills` gains `ly:board-policy`.
   - Pinned by `TestStart_NoStrandLaunchesAndSpawnsWatcher` (`start_test.go:123`). It asserts exact equality of the list.
   - Not pinned: `orchcli/watch.go:126` passes `orchSkills` to the watcher. The visible test covers only the start path, so a divergence between the two paths would pass. They share one variable, so I raise nothing.

10. Reed templates: the prime group moves from empty to a full list including `board`.
    - Pinned by `TestLoadConfig_TemplateDefaultsResolve` (`config_test.go`, diff lines 138-148). It asserts the exact prime buttons, `ask` and `ask_default` included.
    - The subtest `BothTemplatesShipTheSameButtons` asserts that both posix and windows templates decode to `wantShipped`.

11. Reconcile path for an existing hub's `reed.yaml`. Its prime group is `[]` or absent, and a group in a layer replaces only that group.
    - **No test in the diff.**
    - The summary names this as an open risk (a hub with a custom beam group sees the new prime set).
    - Whether this is a gap or covered by the existing config-layer tests, I cannot say from my context.

12. Skill prose (`board-policy/SKILL.md`), the orch-role trim, `INDEX.md` and docs.
    - Static prose, exempt.

**Findings**

- **F-T1, LOW, class design, `internal/shuttleengine/open_test.go:54`.** The refusal row cannot catch a mis-ordered check. It uses the default `openerFakeEngine`, which writes nothing in `OpenLaunch`, so the "before anything is created" ordering is not discriminating. Put the skills check ahead of `OpenLaunch` and assert `len(engine.openSpecs) == 0` on that row. Evidence: `open_test.go:54`, `:82-88`, and `open.go:70-78`.

- **F-T2, LOW, class design, `internal/shuttleengine/claudeengine/command_test.go:459`.** I could not confirm the over-limit row asserts that nothing is created. The shared assertions at `command_test.go:473-523` may or may not check that the directory is uncreated on `wantErr`. If they do not, the doc's claim "refuses the open before anything is written" is unpinned. The handler should check lines 473-523. I did not verify them.

- No finding for behaviours 1 and 4 to 10. Behaviour 3 is not a finding either, since the refusal row covers the inversion.

**Tool calls (1 of 6)**
1. `grep` over `command_test.go`, `fakes_test.go` and `shuttlefake.go`: to see how `board: true` is used, whether `openerFakeEngine` has `SkillLoader`, and whether `shuttlefake` does. I had not seen those fixtures.
