# Card 1 — reflect-reentrant

**Edit:**
- `internal/frictionengine/reflect.go`
- `internal/frictionengine/deps.go`
- `internal/frictionengine/doc.go`
- `internal/frictionengine/reflect_test.go`
- `internal/testkit/shedfake/shedfake.go`
- `internal/loomengine/config.go`
- `internal/loomcli/wiring.go`

**Uses:**
- `internal/shuttleengine#Runner.Attach`
- `internal/friction#ReportFileName`
- `internal/testkit/shedfake#Shuttle`

**Intent:**
Make `frictionengine.Reflect` re-entrant across a killed driving process (D5), so a re-invocation never has a second agent file notes a first one already covers.

`Shuttle` gains `Attach(shuttleengine.Spec) (shuttleengine.Result, bool, error)`, the method `*shuttleengine.Runner` already has, so the compile-time assertion keeps holding.
`shedfake.Shuttle` already carries `Run` and `Attach`, so it becomes the fake `frictionengine.Shuttle` tests use; `shedfake`'s package doc stops claiming `MergeShuttle` satisfies `frictionengine.Shuttle`.

A covered-notes record, a non-`.md` file named once by an unexported constant (`reflection-covered.json`), sits in the friction directory beside the report and holds the sorted note file names one reflection covers.
`scanNotes` never reads it, because it reads only `.md` entries.

`Reflect` after `validateDeps`, in order:

1. Settle a prior reflection when a record exists.
   Build the reflection spec over the recorded notes and probe `deps.Shuttle.Attach` with it.
   A live agent found: wait on it through `Attach`; `OutcomeDone` archives the covered set, any other outcome returns `StatusFailed` and leaves notes, record and report in place.
   An `Attach` error returns `StatusFailed` without spawning, because a probe that cannot answer may be hiding a live agent.
   No live agent and the report exists: archive the covered set without spawning.
   No live agent and no report: delete the record; its notes are reflected again below.
2. With no record, a leftover report is stale and deleted, as `deleteStaleReport` does today.
3. Scan the remaining notes.
   None: return `StatusReflected` when step 1 archived, `StatusSkipped` otherwise.
   Some: write the record naming exactly these notes, then build the spec and spawn through `deps.Shuttle.Run`; `OutcomeDone` archives the covered set, any other outcome or a `Run` error returns `StatusFailed` with notes and record left in place.

One call spends at most one `deps.Timeout`, measured from entry through `deps.Clock`, so D1's bound on the row and D2's on the halt envelope hold:
`Attach` gives an attached run a fresh deadline, so a spawn after step 1 waited on a live agent gets only the remaining budget as its `Spec.Timeout`, and with none left it spawns nothing, returns `StatusFailed` and leaves the remaining notes for the next reflection.

Every failure the record adds returns `StatusFailed` with a nil error and leaves the notes in place, per the Constraint that no runtime failure below Deps validation returns an error: a record that cannot be read, a record write before the spawn, and a spec build on the settle path.
A record that reads but does not parse is logged and discarded like a record with no report and no live agent, so a corrupt record never blocks every later reflection; its notes are reflected again.

Archiving replaces `archiveFrictionDir`: it creates the first free directory of `<ArchivePrefix><timestamp>`, `<ArchivePrefix><timestamp>-2`, … and moves into it only the covered notes, the record and the report.
A covered note already gone is skipped rather than failing the archive.
The friction directory itself is never renamed or removed, so a note written after the record was written stays behind for the next reflection.
`Report.ReportPath` names the archived report of the last archive this call made, and `Report.NoteCount` the notes that archive covered.

What this lets through, and the bound: a re-invocation archives without spawning only on a report, which under D6 the agent writes only after filing succeeded; it spawns at most one new agent, only over notes outside the settled covered set, and the whole call, attach wait and spawn together, stays within one `friction_timeout_min`.
A reflection killed mid-filing leaves no report, so its notes are reflected and filed again; this is the decision record's accepted risk.

`frictionengine`'s package doc replaces the stale-report paragraph with the record/report/attach rules above, and states that the archive moves only covered files.
`loomengine.LoomFrictionArchivePrefix`'s and `LoomFrictionLock`'s doc comments stop describing the directory as renamed away; the lock still lives outside it.
The `loomcli` wiring comment about staging the loom durable directory stops citing a reflection rename's removal of the old path.

Tests in `reflect_test.go` switch to `shedfake.Shuttle`, keep the skip, failure, stale-report and Deps-validation cases, and rewrite the archive assertions for the moved-files shape (the friction directory still exists and holds only uncovered entries).
New cases, each asserting the fake's `Run`/`Attach` calls, the files left in the friction directory and the archive contents:

- a live agent found by `Attach` is waited on and nothing is spawned;
- a record plus a report with no live agent archives the covered notes and spawns nothing;
- a note written after the record is left behind by that archive and is the only note the next spawn's prompt lists;
- a record with no report and no live agent is discarded and its notes are reflected by a new spawn;
- a note written during the spawn (through the fake's `DuringRun`) stays in the friction directory after the clean-return archive;
- two archives in the same clock second land in distinct directories;
- an `Attach` wait that spends the whole budget (a fake clock advanced in `DuringAttach`) leaves a newer note unspawned and returns `failed`, and a partial spend spawns with the remaining budget as `Spec.Timeout`;
- an unparsable record is discarded and its notes are reflected by a new spawn;
- an unwritable record before the spawn returns `failed` with a nil error and spawns nothing.

**ImpactSummary:** `frictionengine.Shuttle` gains one method that its one production implementer already has; only `loomcli.reflectFriction` calls `Reflect`, and the remaining edits are doc comments.
