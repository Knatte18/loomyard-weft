```yaml
lens: affected-set
outcome: reported
```

Task: affected-set lens. For each change a card makes, find everything outside the card that reads or asserts it, and check which cards cover it.

**Per change: what reads it, and which cards cover it**

1. **Per-task key list gains `beam.run_watch` (card 1).**
   - Covered by card 1: `internal/beamsetting/beamsetting.go:2` package doc, `internal/dobbyengine/tasksetting.go:1`, `tasksetting_test.go:1`, `config.go:57` (`Config` doc), `config.go:139` (`CheckLocalValues` doc), the `internal/dobbyengine/doc.go:138` bullet, and `pattern/PATTERN-config-layers.md:125`.
   - Missed by every card: `internal/boardcli/cli.go:587`, inside `storeVerbs` (declared at :558). The `run_settings` help still says "beam.pull_request, beam.land_notice and beam.autonomy set the task's per-task beam settings".
   - Not falsified: `plugins/ly/skills/board/SKILL.md:266`, `boardengine/doc.go:131,137` and `contracts/orchrole/role-beams.md:56` each name one existing key and don't claim a complete list. `internal/boardcli/help_test.go:62` asserts `beam.land_notice` with `mustContain`, so adding a key doesn't break it.
2. **beam.yaml template gains `run_watch`, `gate_hold_factor`, `gate_hold_min` (card 1).** Readers are the `config_test` `defaultsWith`/`TestLoadConfig` rows, `TestLookupTaskSettings`, and the `configreg` load check. Card 1 covers all of them. No settings golden lists beam keys: a grep for `adopt_fix_tries` found only `config.go`, the tests, `doc.go` and `template.yaml`.
3. **`gateslot.Holder` gains `Marker` (card 5).**
   - Readers: `internal/gatecli/status.go:60`; `internal/fabriccli/anchordeps.go:232`, which formats only Slot, Worktree, Site and PID; `pool_test.go:172-374`; `cardverify/runner_test.go:188,216`; `gatecli/copies_test.go:138`; `gatecli/test_integration_test.go:696`.
   - All of them test only length or emptiness, or read named fields, so none breaks. Card 5 covers the doc in `gateslot/doc.go` and `TestHolders`.
4. **`RunView` gains 3 fields and `ChildReading` gains 1 (card 2).** The only production writers are `DobbyRuns` (`dobbyreads.go`) and `readNoticeChild` (`dobbynotice.go:110`), both on card 2. Zero-value literals in tests stay valid.
5. **`State.Watches` and the `clearDoneRuns` hold (card 3).** `donerun.go` and `TestClearDoneRuns` are covered by card 3. The `state.go` readers are tolerant (omitempty map).
6. **New notice prefix `[run watch]` and the gate-hold `[dobby notice]` condition (cards 4, 7).**
   - Covered: `contracts/orchrole/orch-role.md:81-92` (cards 4 and 7). The hub's `orch-template-role.md` is outside the repo and the overview acknowledges it.
   - Missed by every card: `docs/overview.md:850`. It lists what dobby "sends the nearest live orch" as its own notices plus the `[dobby notice]` lines "of a running child (a dead driver, an API-error stall, a usage limit, idle agents)". The new watch lines and the gate-hold notice, which isn't about a running child, aren't in that list.
7. **New posts through `NotifyRunFrom` with empty or new tags (cards 4, 7).** `TestHubDobbyDeps_RunNoticePostCarriesTheRunsAutonomy` and `TestRunNoticeTags` are covered by cards 4 and 7. No other reader of `runNoticeTopicPrefix` was found in the pre-read.

**Findings**

- **MEDIUM — consistency — card 1 (`_lym/plan/01-run-watch-setting.md`).** Card 1 updates part of the set of texts that list the per-task beam settings. It misses the board upsert help in `internal/boardcli/cli.go:587` (in `storeVerbs`, `cli.go:558`), which names exactly three per-task keys settable through `run_settings`; `run_watch` makes that false. Evidence: `internal/boardcli/cli.go:586-587`.
  - Fix: add `internal/boardcli#storeVerbs` under card 1's `**Edit:**` and have its Intent name `beam.run_watch` in that help line. Confirm the line's span with `lym quarry resolve` and the target with `validate-plan --overlay`.
- **LOW — consistency — cards 4 and 7 (`_lym/plan/04-run-watch-wiring.md`, `07-gate-hold-wiring.md`).** `docs/overview.md:850` lists dobby's lines to the orch. It names neither the `[run watch]` lines nor the gate-hold `[dobby notice]`, which isn't a running-child condition, so the list goes stale. Evidence: `docs/overview.md:850`.
  - Fix: add `docs/overview.md` to card 4 (or as a `Prosa` group) and name the watch lines and the gate-hold notice there.

**Tool calls**
1. Grepped the repo for every file naming `land_notice` or the per-task key list, to find readers outside the pre-read.
2. Read the `land_notice` mentions in SKILL.md, boardengine/boardcli docs and `help_test.go`, plus the notice-prefix mentions in docs. I needed to know whether they enumerate keys or prefixes.
3. Grepped `Holders()` readers and read `boardcli/cli.go:580-592`. The pre-read lacked the Holder consumers and the help text.
4. Read `gatecli/test_integration_test.go:690-712` and `fabriccli/anchordeps.go:225-240` to see whether they pin Holder's full shape.
5. Found the declaration enclosing `boardcli/cli.go:587` and read `docs/overview.md:846-852`, to name the fix glyph and quote the overview line.
