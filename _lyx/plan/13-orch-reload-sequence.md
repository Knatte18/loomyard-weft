# Card 13 — orch-reload-sequence

**Edit:**
- `internal/orchengine/watcher.go`
- `internal/orchengine/state.go`
- `internal/orchengine/doc.go`
- `internal/orchcli/session.go`
- `internal/orchengine/watcher_test.go`
- `internal/orchengine/watcher_compact_test.go`

**Uses:**
- `plan:internal/shuttleengine#Runner.ReloadPlugins`

**Intent:** D9 and D10, watcher half: every reload sequence starts by typing `/reload-plugins`, and only the reload after `/clear` runs the skills and retry steps.
Claude Code re-injects the skills a session invoked after a compaction (A2), so retyping them after a compaction costs a turn and changes nothing; `/clear` loses them.
`/reload-plugins` lets a skill deployed after the session started load at the next reload (#393, #390).

Seam: add `ReloadPlugins(guid string) error` to `orchengine.Session`, and `runnerSession.ReloadPlugins` delegates to `Runner.ReloadPlugins`; the `fakeSession` records it as `"reload-plugins"` in `calls`.

State, in `state.go`:

- A new step constant `ReloadStepPlugins = -2`, first in the sequence, beside `ReloadStepSkills`, `ReloadStepRetry` and `ReloadStepPointer`, with the `ReloadStep` field comment naming it.
- A new field `ReloadSkipsSkills bool` (`reload_skips_skills`): true for a reload entered after a compaction, so the step after the plugins step is the pointer; false after `/clear`.
  `toIdle` and `ResetForFreshLaunch` reset it with the other reload fields.

Watcher, in `watcher.go`:

- `startReload` takes whether the sequence skips the skills step, persists it in `ReloadSkipsSkills`, and enters at `ReloadStepPlugins`.
  The `/clear` path (`tickClearing`) passes false; `reloadAfterCompaction` and `startAutoReload` pass true.
- `reloadStep` normalises `ReloadStepPlugins` as its own step, and normalises `ReloadStepSkills` to the pointer step when `ReloadSkipsSkills` is set.
- The plugins step is typed only on a tick whose idle probe passed: persist `ReloadTypedAt` and the events offset first (as `typeReloadStep` does), call `Session.ReloadPlugins`, then persist the move to the next step (the skills step, or the pointer when `ReloadSkipsSkills`) with `ReloadTypedAt` zero, and return without typing anything else on that tick.
  It ends no turn and starts none, so nothing waits for a turn end; the next step is typed on a later tick whose idle probe passed.
- A restart before the move finds the plugins step persisted and types `/reload-plugins` again once the idle probe passes; the command is idempotent.

Docs, in `doc.go`'s "The reload sequence" section: replace the statement that a clear, a compaction and an auto-compaction each lose the session's skills with D9's premise (a compaction keeps the skills the session invoked, `/clear` loses them); describe the plugins step as the first step of every entry point, persisted as `reload_step` -2 so a restart resumes at it, typed only on a passing idle probe and followed on a later passing tick; say the skills and retry steps run only after `/clear`; and list `reload_skips_skills` with the other persisted fields.

Bound, stated there: only the compaction entries skip the skills step, `/clear` keeps it, and the pointer step still has the session read its role file; a session that did lose a skill misses it until its next `/clear` or restart (R4).

Tests, through `Watcher.Tick`:

- `watcher_test.go`: `TestWatcher_ClearCycleReloadsSkillsThenPointer` asserts the calls `reload-plugins`, then the skills load, then the pointer, with the skills typed on a later tick than `reload-plugins` and only after a passing idle probe; a tick whose idle probe fails after `reload-plugins` types nothing; `TestWatcher_AutoCompactionReloadsSkillsRoleAndPointer` asserts `reload-plugins` then the pointer and no skills load; `TestWatcher_ReloadRestartRetypesOnlyTheUnconfirmedStep` gains a restart at the plugins step that retypes `/reload-plugins` and then moves on.
- `watcher_compact_test.go`: `TestCompact_CycleReloadsSkillsThenPointerNamingTheNote` asserts `reload-plugins`, no skills load, then the pointer naming the cycle's note, and is renamed to say so.

**ImpactSummary:** The orch watcher's reload sequence gains a first plugins step and drops the skills step after compactions; the Session seam gains one method implemented by the orch adapter and the test fake.
