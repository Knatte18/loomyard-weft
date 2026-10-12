---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: internal/pauserecord/record.go:100
    summary: >-
      The parent's pause record rewrites record.json with json.MarshalIndent and a hand-rolled atomic write outside internal/state, so a long-lived older binary drops the keys a newer binary wrote, which PATTERN-tolerant-records forbids
    fix: >-
      In internal/pauserecord/record.go replace the body of Update (the MkdirAll, lock.AcquireWriteLock, Read, json.MarshalIndent and writeAtomically sequence) with one state.UpdateJSON[Record](path, path+".lock", func(current Record, found bool) (Record, error) { ... }) call whose mutate runs the caller's mutate on the record, then sets record.Resumed = withoutExpired(record.Resumed, now), and have Update return the mutated record. Keep Read as it is, since a read-only decode is allowed. UpdateJSON holds the same path+".lock" write lock and keeps the unknown keys of the replaced file, and it aborts on a decode failure before mutate runs, as Read does today. Delete writeAtomically from record.go and give writeMarker in hold.go its own atomic write or a state.WriteJSON call, since a marker is written fresh and never rewritten. Then admit internal/state in the three places that state the import set: the import allowlist in internal/pauserecord/leaf_enforcement_test.go (add github.com/Knatte18/loomyard/internal/state), the pauserecord section of pattern/PATTERN-leaf-packages.md, and the last paragraph of internal/pauserecord/doc.go. Check first that internal/state imports only the standard library, internal/fsx and internal/lock, so no cycle forms. Add one row to the existing record_test.go update test: a record.json carrying an unknown top-level key and an unknown key on an entry that Update leaves untouched still has both keys after Update.
---

### [BLOCKING] Pause record rewrite bypasses internal/state and drops unknown keys

**Location:** `internal/pauserecord/record.go:100` (`Update`), `internal/pauserecord/record.go:120`, `internal/pauserecord/record.go:132` (`writeAtomically`).

**Issue:** `PATTERN-tolerant-records` (added by the task) says a rewrite of a Go-written JSON record goes through `internal/state`, or through a type that keeps unknown keys itself, as `board.json`'s `storeFile` does.
The parent's new `pauserecord.Update` reads `record.json` with `json.Unmarshal` into `Record`, runs the caller's mutation, and replaces the file with `json.MarshalIndent(record)` and its own temp-file rename.
`Record`, `Entry` and `Session` declare no unknown-key carrier and implement no `json.Marshaler`, so every key of the replaced file that this binary's types do not declare is lost on each `Update`.
The decision record's goal is exactly this loss: a run record written by a newer binary must never lose data in a long-lived process started on an older one.
The pause record is rewritten by long-lived processes, the orch watcher (`internal/orchengine/pause.go:164`, `:199`) and the pause and resume verbs (`internal/pauseengine/pause.go:89`, `:215`, `internal/pauseengine/resume.go:49`), so a watcher started on an older binary would drop a field a newer `lyx pause` added to an entry or to the file, on its next `resume_noticed` update.
The scan in `cmd/lyx/tolerantrecords_test.go` only looks for `DisallowUnknownFields`, so it does not see this bypass, and `pauserecord.Read` itself is tolerant; the defect is the rewrite alone.
The read side, `ListNetOps` and `writeMarker`, is fine: a marker is written whole once and deleted, never rewritten.
`pauserecord` currently admits only the standard library and `internal/lock` (`pattern/PATTERN-leaf-packages.md`, `leaf_enforcement_test.go`), which is why the parent did not use `state`; the leaf set has to widen by `internal/state`, which itself imports only `fsx` and `lock`, so the lock the package already takes is the one `state.UpdateJSON` takes.

## Rejected

- Strict or direct-write run-record access in the parent's `internal/shedverbs/loop.go`, `internal/shedengine/{goto,rename,run}.go`, `internal/battencli/{wire,arm}.go` and `internal/loomcli/{resume,start,looparm,landingdeps,wiring}.go`: every `shedengine.Status` read uses `state.ReadJSONNoMkdir` and every write uses `state.UpdateJSON`; the `json.Unmarshal` at `loop.go:408` decodes a step envelope map and rewrites nothing, and `loop.go:482` writes a joined log. No `ReadJSONStrict` or `DisallowUnknownFields` remains anywhere.
- `internal/alignment/session.go`: `Begin` and `failBegin` still write `attempt.json` through `WriteAttempt`, which calls `state.WriteJSON`, and the parent's pause handling (`observe`, `awakeSpent`) reads only the pause record and never touches `attempt.json`, so the task's keeping holds after the merge.
- `internal/websterengine`: the parent's removal of the pause flag, `ErrPaused` and `outcomePaused` leaves `doc.go`, `runlevel.go`, `beginbatch.go` and `verifygate_test.go` agreeing with the task's no-rerun-after-hang gate; `outcome_test.go` still pins `outcome: paused` as rejected.
- `PATTERN.md` and the changed `pattern/` files: the task's gate watchdog and the parent's pause hold are both listed in `PATTERN-daemon-wakeups.md` and in `unboundedWaitFiles` in `cmd/lyx/daemonwakeups_test.go`; the merged `test-budget.yaml` rows are the sum of both sides' edits and the verify gate passed on the merged tree, so no budget finding.
