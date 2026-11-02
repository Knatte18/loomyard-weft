```yaml
lens: existing-paths
outcome: reported
```

Lens review (existing-paths): what each decision meets in the code that already runs.

**Findings**

- **BLOCKING — D4 / D1 (the fixer's own gate call joins the run that is waiting on it).**
  - The fixer's final "whole gate call" is the same `lym gate test --gate card --card <id>` from the same worktree directory.
  - `detachKeyDir(anchor, verb, runDir, words)` hashes only the verb, the call's directory and the words; there is no strand in the hash. Its doc says "Calls with the same key share one current run" (`internal/gatecli/detachfront.go`).
  - So the fixer's call joins the fork's detached run, and that run is blocked waiting on this same fixer. They deadlock until `fixer_timeout_min`.
  - D4's bound ("a `gate-fixer` strand never hands off") stops a nested spawn but not this join. The record says nothing about the fixer's run being keyed apart from its caller's.

- **BLOCKING — D2 / D3, Plan-Write (the fork-mode plan gate gets no fixer).**
  - `planWriteGateSpec` (`internal/shedrecipe/entries_planwrite.go`) calls `resolveGateSpec` and only then sets `spec[i].Forked = loomshed.NewLensReviewPlanGate(...)` on the `plan` entry.
  - The record's note puts the wrap in `resolveGateEntry`, and D2 says it wraps "`Forked` when set". At that point `Forked` is not set yet, and the later assignment replaces it with an unwrapped closure.
  - `tapestry.plan_lenses` defaults to `twill-plan`, so every default Plan-Write run takes the forked closure and never reaches the fixer. The record says nothing about this.

- **MEDIUM — D2 (earlier entries' verdicts go stale after the fixer).**
  - `evaluateGate` runs the entries in order and memoises the verdict for the arrival.
  - On Darn-Write the `clean` entry passes before the wrapped `verify` entry spawns the fixer. The wrapped closure then re-runs only `verify`, and the memoised verdict, which finalize also uses, never re-checks `clean`.
  - A fixer that leaves untracked or uncommitted files therefore passes the row. D5 asks the fixer to leave the tree clean, but nothing in Go checks it. That contradicts "Go's re-run judges, never the fixer" for any entry earlier in the list.

- **MEDIUM — D1 / D8 (an orphaned row fixer outlives cancel and goto).**
  - The row fixer's record sits only in the row's `.lym` run scratch directory.
  - Webster's fixers are held in webster state, which its reset and cancel read and stop (`ownerlessRunWarnings`, the `VerifyFixers` records).
  - Suppose the driver dies mid-fixer and the operator then runs `lym loom goto` to another row, or `lym batten cancel`, instead of resuming. No path reads the gate-fixer record, so the strand keeps editing the worktree under a different row.
  - runproc ends only registered processes by role (`internal/runproc/doc.go`), and the record names no role or owner registration for the fixer. D1 covers attach on restart only.

- **LOW — D2, Darn-Write kept findings.** `KeepGateFindings` keeps the final findings text, and `renderPriorWork` points a resumed writer at it. With D2 that text now carries the fixer's reply or dead note. This survives and helps, but the record doesn't mention it.

**Paths traced, per decision**
- **D1:**
  - Webster `VerifyFixers` state with reset and cancel: survives (D6 keeps it).
  - Row fixer under goto/cancel after a driver crash: breaks (finding above).
  - Detached run identity: breaks (first finding).
- **D2:**
  - `darnWriteEntry` stamping `Reprompt` after resolve: survives.
  - `planWriteGateSpec` setting `Forked` after resolve: breaks.
  - `evaluateGate` order and memo with `clean` before `verify`: breaks.
  - The finalize evaluation reading the memo: misreads a stale `clean` pass.
  - `DarnGateReprompt` line: survives.
- **D3:**
  - `applyVerifyAttempts` (`internal/loomrecipe/darnrecipe.go`) changes `entry["attempts"]` in place on the element map, so a `fixer` key on that element survives.
  - The `gateEntryKeys` closed list must admit `fixer`; the record already calls it a new key, so this is plan-level.
  - gate.yaml template key merge: survives.
- **D4:**
  - Detach key sharing: breaks.
  - Lease renewal by the fork's keep-warm re-calls: survives.
  - The fork's `self_fix_cap`: survives.
- **D6:** fork-first ladder and fix-verify records: survives.
- **D7:** the Discussion lens-review gating text is covered by D7: survives.
- **D8:**
  - Kept gate findings: survives.
  - Gate span recorded around the blocking closure: survives.

**Tool calls**
1. Grepped `detachfront.go` and `detachrun.go` for how the detach key and run dir are formed; my context lacked the key derivation.
2. Grepped `verify_attempts` stamping, `RunScratchDir` writers and `Env.Setting`; my context lacked the stamp site.
3. Grepped runproc `EndRole` users and the gateslot gate-key decoding; my context lacked how strands and processes are ended.
4. Read the `detachKeyDir` callers and `planWriteGateSpec`; I needed whether the key includes the strand and when `Forked` is set.
5. Read `applyVerifyAttempts` in `internal/loomrecipe/darnrecipe.go`; I needed whether stamping rebuilds the gate element.
6. Read the runproc doc on roles and ending; I needed whether anything ends an unregistered strand.
