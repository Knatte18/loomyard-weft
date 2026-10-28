```yaml
lens: affected-set
outcome: reported
```

Affected-set lens: what each card's change forces outside the card, and which cards cover each set.

**Per change: the set found, and the cards that cover it**

- **Quoted `MessageLine`, reminder and `read` header (card 3).**
  - Set: `creelengine/reminder.go` and `doc.go`, `verbs_test.go`, `deliver_test.go`, `server_test.go:971` (in `TestDaemon_Lifecycle`), `creelcli/hookread.go` Long text, `cli_test.go:606`, `readtext_test.go`, `internal/orchcli/doc.go:78`.
  - The two smoke tests match only the prefix, so they need no change.
  - Covered by cards 3, 12 and 17. Complete.
- **`ToolBoundaryContext` removed and the capability re-shaped (card 12).**
  - Set: `shuttleengine/engine.go:219-220`, `claudeengine/capabilities.go:35-37`, `creelcli/hookread.go:71`, `cli_test.go:606`, `shuttleengines_test.go:344-347`, `claudeengine/settings_test.go:203` (`TestHookContext`), `claudeengine/doc.go:129`.
  - Also the `capabilities.go` file header ("tool-boundary context rendering").
  - Covered by card 12, except the `capabilities.go` header.
- **`--append-system-prompt` replaced by `--append-system-prompt-file` (card 16).**
  - Set: `command.go` builders, `command_test.go` rows (233, 244, 296, 308, 562, 589), `prepare_test.go:292-314` (`TestPrepare_AppendSystemPromptMatchesDenyNotice`), `settings_test.go:812` (keep reason), `claudeengine/doc.go`, `docs/overview.md`.
  - Also the `prepare_test.go:3` file header, which names "the --append-system-prompt notice".
  - Covered by card 16, except the `prepare_test.go` header.
- **Stop renders two entries in a hub (card 14).**
  - Readers: `settings_test.go:228` (`TestBuildSettings_CreelMailHook`, card 14), `settings_test.go:326` and `:703` (both standalone, unaffected), `TestBuildSettings_MatchesGolden` (standalone).
  - Covered by card 14. `hookcmd_integration_test.go:35`'s `buildSettings` call was not checked.
- **Line cap at `post` (card 4).**
  - Production posters: loomcli decision notice (card 5), `orchcli/notify.go:220,232` batten notices, `orchengine/watcher.go:612` handoff stencils (569 and 933 bytes), `orchengine/pause.go:255` `pauserecord.ResumeLine` (short).
  - None but loomcli's notice can reach 8 KiB, so card 5 covers it.
- **`IdleProbe` gains `NoTurn` and `TurnEndedAt` (card 7).**
  - Probe literals outside shuttleengine: `orchengine/watcher_test.go`, `creelcli/delivery_test.go`. Both are stub inputs, and new zero fields break neither.
  - Covered by cards 7 and 8.
- **`creelengine.Strand` gains `CreelHooks` and `PaneMail` (card 8).**
  - `worktreeStrands` and `creelfake` are covered by card 8.
  - Strands seeded with no provider stay zero, so `TestHubDelivery_SkipsAWorktreeThatNoLongerResolves` is unaffected.
- **The reminder and pane notice end for marked Claude strands, the orch included (cards 9 to 11 and 17).**
  - Set: orch stencil step 4 and `orchcli/doc.go:78` (card 17), `docs/overview.md` (card 17), `SANDBOX-CREEL-SUITE.md` (card 17), `creelengine/doc.go` (several cards).
  - Missed: `internal/orchengine/doc.go:386`, `:406` and `:423`, and `internal/orchengine/pause.go:235`.
  - `orchcli/creelseam.go:159` reads `reminded` entries, which still occur for pane-mail strands, so it stays accurate.

**Findings**

1. **MEDIUM, consistency, card 17 (D11 doc set).**
   Card 17's rewrite of the docs that describe the reminder flow misses `internal/orchengine`.
   - `internal/orchengine/doc.go:386`: "keeps the orch's other mail unreminded until it returns to idle".
   - `internal/orchengine/doc.go:406`: "old mail it posted is reminded".
   - `internal/orchengine/doc.go:423`: "posted as creel mail to the orch strand's name, which creel reminds once the pause is lifted".
   - `internal/orchengine/pause.go:235`, the `Watcher.queueResumeLine` comment: "Creel reminds mail once the pause is lifted".

   The orch strand is a marked Claude strand, so after this task its mail, `pauserecord.ResumeLine` included, arrives through its hooks or the bell, never a reminder.
   The decision record's Scope covers "the docs and stencils that describe the old reminder flow".
   Fix: add `internal/orchengine/doc.go` and `internal/orchengine#Watcher.queueResumeLine` to card 17's Edit list.
2. **NIT, consistency, card 12.**
   `internal/shuttleengine/claudeengine/capabilities.go:2`, the file header, lists "tool-boundary context rendering" among Claude's capabilities.
   Card 12 deletes `ToolBoundaryContext` and renames what the capability renders, but does not cover this header.
3. **NIT, consistency, card 16.**
   `internal/shuttleengine/claudeengine/prepare_test.go:3`, the file header, says the file covers "the --append-system-prompt notice".
   Card 16 replaces that flag with `--append-system-prompt-file` and edits the test, but not this header.

Tool calls:
1. Grepped for quoted-line, `append-system-prompt`, `ToolBoundaryContext` and `hook-read` readers repo-wide, since the manifest held only the creel packages.
2. Located those hits by line and listed each file's test functions, to map them to the cards' named tests.
3. Grepped `buildSettings` hub calls, Stop-count asserts, `IdleProbe` literals outside shuttleengine, creel posters and reminder mentions in orch code, which no manifest file covered.
4. Read the `TestPrepare_WritesArtifacts` Stop assert, `creelseam.readAtOf`, `orchengine/pause.go`'s `queueResumeLine` comment and the orchcli notify posts, to judge those hits.
5. Read `engineWith` and the handoff stencil sizes, to check whether the Stop-count assert runs in a hub and whether any orch post can pass 8 KiB.
6. Read `pauserecord.ResumeLine` and its users, to judge the orch resume-line post under hook delivery.
