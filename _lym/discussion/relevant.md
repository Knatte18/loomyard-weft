# Relevant files

- `internal/dobbyengine/runnotice.go` `RunNoticeDeps`, `ChildReading`, `RunNoticeState`, `deliverRunNotice`, `renderRunNotice` — the running-child notices whose child reader, memory and delivery the watch mirrors
- `internal/dobbyengine/passes.go` `RunView` — the per-run view the dobby reads each tick, with the derived run state and batten's state and row
- `internal/dobbyengine/template.yaml` — beam.yaml's keys, where `run_watch`, `gate_hold_factor` and `gate_hold_min` join
- `internal/dobbyengine/tasksetting.go` `LookupLandNotice`, `lookupTaskSetting` — the one per-task setting lookup `run_watch` is read through
- `internal/dobbyengine/doc.go` — the dobby's doc of its tick and notices, which the watch joins
- `internal/beamcli/dobbydeps.go` `runNoticeTags`, `requirePosted` — the dobby's wiring of its notice post seams
- `internal/battenshed/notice.go` `noticeStep`, `noticeCondition`, `stateChangeReady` — batten's halted-child notice and its holds, which stay as they are
- `internal/loomcli/rownotice.go` `rowNoticeProducer` — the `notify_parent_on` row line, left unchanged
- `internal/runfsm/state.go` `Derive` — the derived run states the watch reports
- `internal/gateslot/pool.go` `Holder`, `Pool.Holders` — the slot holder records the gate-hold watch reads
- `internal/gateslot/config.go` — the tier `timeout_min` the bound derives from
- `internal/proc/proc_linux.go` `TreeSample` — the process tree the gate-hold notice names
- `internal/gatecli/processcap.go` `runMarker` — the environment marker every process of a tier run carries, which the slot holder records
- `internal/proc/envmarker_linux.go` `PIDsWithEnv` — finds a gate run's orphaned processes by its marker
- `internal/pauserecord/sleep.go` `AwakeSince` — awake time over the dobby's recorded sleeps
- `internal/creelengine/verbs.go` `PostTags` — topic replacement and the run tag the watch lines leave unset
- `contracts/orchrole/orch-role.md` — the orch's Notices section the new lines join
