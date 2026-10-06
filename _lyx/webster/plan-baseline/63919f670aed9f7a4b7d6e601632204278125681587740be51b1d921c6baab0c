# Card 3 — orch-reload-one-turn

**Edit:**
- `internal/orchengine#Session`
- `internal/orchengine#State`
- `internal/orchengine#ResetForFreshLaunch`
- `internal/orchengine#Watcher.toIdle`
- `internal/orchengine#Watcher.startReload`
- `internal/orchengine#Watcher.typeReloadStep`
- `internal/orchengine#Watcher.advanceReload`
- `internal/orchengine#Watcher.skipSkill`
- `internal/orchengine#Watcher.tickResuming`
- `internal/orchengine/watcher_test.go`
- `internal/orchengine/watcher_compact_test.go`
- `internal/orchengine/state_test.go`
- `internal/orchcli/session.go`

**Prosa:**
- `internal/orchengine/doc.go`

**Uses:**
- `plan:internal/shuttleengine#Runner.LoadSkills`
- `plan:internal/shuttleengine#Runner.ClassifySkillLoad`
- `internal/shuttleengine#SkillLoadReport`

**Intent:**
Shrink the orch reload sequence to a skills step, an optional retry step and the pointer step, on the per-list primitives card 2 added (Decisions 5, 6, 7, 11, 12; issue #394 on the orch side).

The `Session` seam swaps `LoadSkill` and `SkillUnknown` for `LoadSkills(guid string, skills []string) error` and `ClassifySkillLoad(turnEnd shuttleengine.Event, skills []string) (shuttleengine.SkillLoadReport, error)`.
`internal/orchcli/session.go`'s `runnerSession` drops its two old methods and delegates the new ones to `Runner.LoadSkills` and `Runner.ClassifySkillLoad`, with a debug log on the send like its siblings.

`State` gains `ReloadRetry []string` (`reload_retry`), the skills the retry step still loads.
`ReloadStep` names a step, not a skill index: the skills step is 0, the retry step is a value no old per-skill index can take (e.g. -1), and every other value is the pointer step.
A persisted retry step with an empty `ReloadRetry`, and every old index past 0, are read as the pointer step (Decision 12), so an in-flight reload across the upgrade skips at most that cycle's remaining skills.
`ResetForFreshLaunch` and `toIdle` clear `ReloadRetry` with the step.

The steps, each typed only on a tick whose idle probe passed (`PATTERN-orch-pane-single-writer`):

- The skills step types `LoadSkills(w.skills)`; an empty skill list starts the sequence at the pointer step.
  Its first turn end after the typing (`w.seen.firstTurnEnd`) is classified with `ClassifySkillLoad`.
  `Unknown` skills are skipped with `logger.Warn` naming each and cause `unknown`;
  an unverified report logs one `skill load unverified` Warn listing the skills and goes to the pointer, so an unknown skill on that turn goes undetected and shows only in the Warn's list, with no stall;
  a non-empty `Missing` list moves to the retry step, writing the step and `ReloadRetry` in one save, else the pointer step follows.
  Past the handoff timeout with no turn end, every skill is skipped with cause `timeout`, with no retry.
- The retry step types `LoadSkills(st.ReloadRetry)` and classifies its turn end against `ReloadRetry`: `Unknown` with cause `unknown`, still `Missing` with cause `not loaded`, unverified with the one Warn, and a timeout with cause `timeout`; then the pointer step.
- The pointer step is unchanged.

Each step keeps today's persistence: its first typing time and events offset are saved before the text is typed, a restarted watcher re-sends only an unconfirmed step once the probe passes, and never restarts its timeout.
A watcher restarted before the move into the retry step reads the same turn end again and classifies it to the same result; one restarted after it never retries twice.

`watcher_test.go`'s `fakeSession` records `skills:<a,b>` per `LoadSkills` call and answers `ClassifySkillLoad` from a script keyed by the turn end's message, defaulting to all loaded.
The existing reload tests change to the new step list rather than gaining parallel ones:
`assertReload` expects one skills call then the pointer;
`TestWatcher_SkillSkipCauses` becomes rows for unknown skipped with no retry, missing retried once naming only it, still missing after the retry skipped, an unverified report going straight to the pointer, and a timed-out skills step skipped with no retry;
the restart tests pin that an unconfirmed skills step is re-sent with its first typing time kept, that a restart after the move into the retry step never sends a second retry, and that a restart in the retry step re-sends only the retry;
a new row pins that a persisted old-style `reload_step` past the skills step resumes at the pointer step.
`watcher_compact_test.go`'s reload test and `state_test.go`'s reset row follow (`ReloadRetry` cleared).

`orchengine/doc.go`'s "The reload sequence" section is rewritten to the skills step, the optional retry step and the pointer step, with `reload_retry` among the persisted fields and the old-step rule.

**ImpactSummary:** Changes the orch `Session` seam, its one production adapter in orchcli and the resuming phase's step machine; persisted `State` gains one field and reads old reload steps as the pointer step.

**Commit:** `3: orch: reload skills in one turn, retry missing once`
