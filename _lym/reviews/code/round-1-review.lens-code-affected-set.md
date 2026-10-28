```yaml
lens: code-affected-set
outcome: reported
```

**Task:** lens code-affected-set. For each change the diff makes, I looked for what else calls, reads or asserts it.

**Per change, the set found and whether it is covered:**

- **gitexec EAGAIN retry (`runSpawning`, `runCommand` seam).**
  - The retry sits in `runCoreWithin`'s three `runAttempt` call sites, so every `gitexec.Run` caller gets it.
  - Doc: `gitexec/doc.go` is updated.
  - Test: `startretry_test.go`, with a seam in `export_test.go`.
  - PATTERN: `PATTERN-gitrepo-client-boundary` is respected. No raw git site is added.
  - Covered.
- **`proc.IsSpawnAgain`, `NoteSpawnAgain`, `LastSpawnAgain`; `isExecAgain` removed from reed.**
  - A repo-wide grep finds no leftover reference to `isExecAgain`, in code, tests or docs.
  - reed's only EAGAIN use is `retryExecAgain`, which now uses `proc`.
  - `proc/doc.go` and `reedengine/doc.go` are updated.
  - Covered.
- **New `host-resources` class (`TransientHostResources`, `ErrAgentDied`, `shedtransient.Class`).**
  - The class is not enumerated anywhere else. A grep for `agent-start` and `TransientAgentStart` finds only its own definition, `shedtransient.go` and `landingshed/fixerrun.go`, which marks an agent-start failure itself and needs no change.
  - `shedtransient`'s import allowlist is updated in the test and in `doc.go`.
  - `PATTERN.md` and `pattern/` do not list the class set or the allowlist (a grep of both finds only the `PATTERN-transient-stop` line), so no PATTERN edit is needed.
  - `shedengine/transient.go` and `shedtransient/doc.go` are updated.
  - `arbiterstop` eligibility keys on a non-empty transient class, so it picks up the new class without a change.
  - Covered.
- **Other `OutcomeDied` mappings: `burlerreview.go:282` and `:452`.**
  - They do not wrap `ErrAgentDied`.
  - The decision record scopes the wrap to the seats adapter, so this is not a finding.
- **Loop `pause_reason` envelope key.**
  - It is written in both loop envelope paths: `loop.go` and `loopwait.go`'s `loopExitStop`.
  - Reader: `sheddriver/driverules.go`, which reads it only when the envelope's own reason is empty and the loop state is paused.
  - Docs: `shedverbs/doc.go:55,61` and `sheddriver/doc.go:45` are updated.
  - The stencils (`contracts/stencils/shed/*`) name `loop.trace_copy`, `loop.stderr_path` and `interrupt_policy`, not an enumerated key list, so none needs an edit.
  - Covered.
- **`AfterInterrupt` hook contract: producer, trace file and ordering.**
  - Callers: `loop.go:329` and `loopwait.go:519`.
  - Implementer: `loomCLI.loomAfterInterrupt`, the only one (`arm.go:343`).
  - The `spec.go` doc is updated, along with `shedverbs/doc.go:40` and `loomcli/doc.go`.
  - On the `loopExitStop` path the hook still gets `loop-exited` and the loop log as before. Nothing there is wrong.
  - Covered.
- **`ClassifyHalt` now maps paused plus interrupted to `interrupted-stop`.**
  - The only producer of `interrupted=true` is `looparm.go`.
  - `anomaly.go` doc comments and `anomaly_test.go` are updated.
  - The halt note format and `internal/haltnote` are untouched, as `PATTERN-sole-parsers` requires.
  - Covered.
- **Declared PATTERN changes.**
  - The decision record declares no PATTERN change, and `git diff --stat` over `PATTERN.md` and `pattern/` is empty.
  - Nothing is owed.
- **Commit messages.**
  - None of the three carries a `Pattern-Departure:` trailer, and none is needed.

**Findings:** none. Every member of the affected set I found is updated or needs no change.

**Tool calls (6):**
1. Grep for `agent-start`, `TransientAgentStart` and `isExecAgain` outside tests and `_lym`: to find other class enumerations or leftover references.
2. Grep for envelope keys, `shedtransient` in PATTERN, and `ErrAgentDied`/`OutcomeDied`: my context lacked the stencil, doc and sibling-adapter references.
3. Grep for `interrupted-stop`, `Interrupted:` and `AfterInterrupt` in Go and docs: to find the hook's implementers and callers.
4. Read `spec.go:78-90` and `loopwait.go:505-525`, and grepped for transient class names and `pause_reason`: to check the second hook caller and the docs.
5. Grep for `OutcomeDied`, `ErrNotStarted`, `execAgain` and `IsSpawnAgain`: to find other `died` mappings and EAGAIN users.
6. Grep for `isExecAgain` in tests and docs, and `git diff --stat` over `PATTERN.md` and `pattern/`: to confirm no stale reference and no PATTERN change.
