# Card 15 — The orch watcher reloads skills, role and note on every path in

**Create:**
- `contracts/stencils/orch/orch-template-reload.md`

**Edit:**
- `contracts/stencils/stencils.go`
- `internal/orchengine/state.go`
- `internal/orchengine/watcher.go`
- `internal/orchengine/prompt.go`
- `internal/orchengine/doc.go`
- `internal/orchcli/session.go`
- `internal/orchcli/watch.go`
- `internal/orchengine/watcher_test.go`
- `internal/orchengine/watcher_compact_test.go`
- `internal/orchengine/state_test.go`
- `internal/orchengine/prompt_test.go`
- `internal/orchcli/watch_test.go`

**Uses:**
- `plan:internal/shuttleengine#Runner.LoadSkill`
- `plan:internal/shuttleengine#Runner.SkillUnknown`
- `plan:internal/shuttleengine#Runner.CompactedSince`
- `plan:internal/orchengine#RenderRoleFile`
- `internal/orchengine#Session`
- `internal/orchengine#Watcher`

**Intent:**
D7: after a clear, after a compaction the watcher ran, and after an auto-compaction, the watcher types each orch skill as its own turn and then the one-line role pointer, so the session never runs on lost skills or a lost role.
`start` and `--adopt` already load through the launch spec (card 12).

Seam: `Session` gains `LoadSkill(guid, skill string) error`, `SkillUnknown(guid, skill string) (bool, error)` and `CompactedSince(turnEnd shuttleengine.Event, since time.Time) (time.Time, bool, error)`; `runnerSession` delegates each to the matching `Runner` method with a debug log, as its other methods do.
`NewWatcher` takes the orch skill list, which `watch.go` passes from `orchSkills` (card 12); the skill-load timeout is the config's `HandoffTimeout()`, the same bound the launch spec carries.

The reload sequence is the resuming phase, extended: it walks the skills in order, then the pointer, one step at a time, persisted in `State` as the step index, the step's first typing time and the pointer line (`PendingResume`, rendered when the phase is entered, after re-rendering the role file).
- A skill step types `LoadSkill` only when the idle probe passed on the same tick, persisting the step before typing.
  It is confirmed by a turn end read after it was typed, and skipped, with a log entry naming the skill and the cause, when `SkillUnknown` reports it or when its timeout, measured from its first typing time, passes.
  Confirm and skip both persist by advancing the step, so a restarted watcher never re-types a confirmed or skipped skill and never gives it a fresh timeout; an unconfirmed step is re-typed after a restart, as the phase's other injections are, and loading a skill twice costs one turn and changes nothing.
- The pointer step types `PendingResume` under the same idle rule and ends the phase at its first turn end, whose context reading becomes the new one, as resuming does today; its timeout runs from the pointer step's first typing time and returns to idle with `resume timed out`.

Entry points:
- after `/clear`: `tickClearing` enters the phase with the resume pointer naming the note, as today;
- after a compaction the watcher ran: `tickCompacting`'s completion enters the phase with the resume pointer naming the cycle's note (`LastHandoff`, card 13) instead of returning to idle;
- after an auto-compaction: in phase idle, at each turn end read, the watcher asks `CompactedSince` with `State.CompactionBaseline`; a boundary found enters the phase with the new one-line `orch-template-reload` pointer (read `{{.role_path}}` and continue the work you were doing; no note), registered in `stencils.go` and rendered by a new single-line render in `prompt.go`.
`CompactionBaseline` is set to the launch time by `ResetForFreshLaunch`, so a boundary an adopted session already carried never triggers a reload, and moves to each handled boundary's time, including a compaction the watcher ran, so no boundary reloads twice.
Every typed line keeps the rule that nothing is typed unless the idle probe passed on the same tick.

Tests (fake `Session`, injected clock): a clear cycle, a compact cycle and an auto-compaction each end with every skill loaded and then the pointer typed, the note named exactly when one exists; a skill reported unknown and a skill whose load times out are each skipped and logged and the sequence goes on; a watcher restarted mid-sequence re-types the unconfirmed step, does not re-type a confirmed or skipped one, and does not restart a skipped skill's timeout; no step types on a tick whose idle probe failed; a boundary older than the baseline triggers nothing.
`orchengine/doc.go` documents the reload sequence, its entry points and its restart rules in the cycle sections.

**ImpactSummary:** the orch watcher's resuming phase, compacting completion and idle phase, plus three Session methods and their adapter; `NewWatcher`'s one production caller is `watch.go`.

**Commit:** 15: orch: reload skills, role and note after a clear, a compaction and an auto-compaction
