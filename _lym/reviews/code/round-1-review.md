---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/driftengine/accepted.go:135
    summary: testFiles admits an empty test name against a top-level dotfile, and a stem match across any extension, so a name that is no baseline test can pass the accept check
    fix: "In `testFiles` (internal/driftengine/accepted.go), match a file when `rel == test`, or when `path.Ext(rel)` is non-empty, `rel` minus that extension is non-empty and equals `test`, so a dotfile such as `.gitkeep` never yields the stem empty string. Refuse an empty `test` at the start of `requireBaselineTest` with the same message naming the test and the directory. Add rows to the name table in `TestAcceptWithdraw` (internal/driftengine/verbs_test.go) for the empty name against a baseline holding `.gitkeep`, and for `t1.txt` against a baseline holding only `t1.txt.bak`, deciding and asserting whether that second name is admitted."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/driftcli/cli.go:149
    summary: acceptTest, the verb's own orchestration of check then capture then accept, has no test
    fix: "Add one table-driven test in internal/driftcli, using the existing worktree and status fixtures of that package's integration tests, that drives `lym drift accept` and asserts: a refusal by `CheckAccept` runs no capture; a capture that does not pass returns the `drift accept refused: the capture of ... did not pass ... way forward` error and leaves the scope record without the acceptance; a passing capture keeps the output, records the acceptance and returns `log` in the result; the capture is handed `scope.Skip`."
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/driftengine/accepted.go:59
    summary: keepAcceptedOutput drops the earlier kept output before it moves the new files, so an I/O failure or a failed scope write leaves kept output that disagrees with the unchanged scope record
    fix: "In `keepAcceptedOutput` (internal/driftengine/accepted.go), move the new files first and remove only an earlier kept file that the new set did not replace, or stage the new set beside `AcceptedTestsDir` and swap it in, so a failure before the scope write leaves the earlier kept output intact. Apply the same ordering in `Withdraw`: drop the kept output only after the scope record is written, or accept that the scope write follows the drop and make a failed drop leave nothing dropped."
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/driftengine/capture.go:272
    summary: concurrent accepts in one worktree share AcceptingDir, which RunAcceptanceCapture empties with no lock and before taking a gate slot
    fix: "In `acceptTest` (internal/driftcli/cli.go), hold one lock file under the drift ephemeral directory, declared in `driftengine.Paths`, from before `RunAcceptanceCapture` until `Accept` returns, so two accepts never share `AcceptingDir`. The lock follows the lock-file pattern of `ReadScope` and `UpdateScope`."
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/driftcli/cli.go:177
    summary: acceptTest reads the run status before a capture that can last minutes and hands that stale status to Accept
    fix: "In `acceptTest` (internal/driftcli/cli.go), call `w.verbStatus()` again after the capture and pass that fresh status to `driftengine.Accept`, so an agent session or row state that changed during the capture decides who the acceptance is by, or refuses it."
  - id: F6
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/driftengine/recheck.go:121
    summary: wording still says the accepted profile was captured at a commit, though the profile is now built and AcceptedProfileHead records the commit a compare against it last passed at
    fix: "Reword to name the compare, not a capture, in `internal/driftengine/recheck.go` at the doc comment on line 36 and the Stuck text on line 121 (for example no compare against the accepted profile passed at the commit of the last compare), in `internal/driftengine/doc.go` at line 25 (the commit a compare against the accepted profile last passed at) and line 148, and in the helper comment at `internal/driftengine/recheck_test.go:47`. Keep any test assertion on the Stuck text in step with it."
---

### [MEDIUM] Name match can admit a name that is no baseline test

**Location:** internal/driftengine/accepted.go:135
**Issue:** `testFiles` matches a file when `rel[:len(rel)-len(path.Ext(rel))] == test`. For a top-level dotfile such as `.gitkeep`, `path.Ext` returns the whole name, so the stem is the empty string and `lym drift accept ""` (which `cobra.ExactArgs(1)` lets through) passes `requireBaselineTest`. The same rule admits `t1.txt` through a file named only `t1.txt.bak`. The decision record's wanted item 1 refuses a name that is not a file under the baseline directory. No test row covers either shape, and the focus directive names both.

### [MEDIUM] The accept verb's orchestration has no test

**Location:** internal/driftcli/cli.go:149
**Issue:** `acceptTest` is new behaviour: it runs `CheckAccept`, reads the capture step and scope, runs `RunAcceptanceCapture` with the scope's skip list, refuses when the capture did not pass, then calls `Accept` and returns the `log` key. The diff touches no `internal/driftcli` test, and `verbs_test.go` and `capture_test.go` test the engine functions only. The order (no capture spent on a refusal), the capture-failure refusal and the absence of an `Accept` call after it are pinned nowhere.

### [LOW] A failure leaves kept output that disagrees with the scope record

**Location:** internal/driftengine/accepted.go:59
**Issue:** `keepAcceptedOutput` runs inside `Accept`'s `UpdateScope` mutate and calls `dropAcceptedOutput` before it moves the staged files. A failed `os.Rename` after the drop, or a failed scope write after a successful keep, leaves the earlier kept output gone or partial while the scope record keeps the old acceptance and its `AcceptedProfileHead`. Drift-Verify then skips the rebuild when HEAD equals that head, or builds from a partial kept set. `Withdraw` has the same shape around `dropAcceptedOutput`. Only an I/O failure triggers it.

### [LOW] Concurrent accepts share AcceptingDir

**Location:** internal/driftengine/capture.go:272
**Issue:** `RunAcceptanceCapture` empties `AcceptingDir` before it takes a gate slot and holds no lock across capture and `Accept`. Two accepts in one worktree can wipe each other's staged output, so one refuses with no output captured or keeps output the other capture wrote. No record is corrupted.

### [LOW] Accept runs on a run status read before the capture

**Location:** internal/driftcli/cli.go:177
**Issue:** `acceptTest` reads `st` once, before a capture that can take minutes, and `Accept` uses it to decide who the acceptance is by. An agent session that closed or a row that changed state during the capture is not seen. The earlier code read the status immediately before `Accept`.

### [LOW] Stale wording from the capture design

**Location:** internal/driftengine/recheck.go:121
**Issue:** The accepted profile is no longer captured at a commit: Drift-Verify builds it and records `AcceptedProfileHead` only when a compare against it passes, as `records.go:103` and `verify.go:200` now say. `recheck.go:36` and `recheck.go:121` (a user-visible Stuck text), `doc.go:25`, `doc.go:148` and `recheck_test.go:47` still say the profile was captured at a commit.

## Rejected

- Lens code-affected-set, `lapse` leaves kept output behind: no reader reads a lapsed test's kept files, and a re-accept replaces them first; the decision record asks for nothing of Drift-Recheck, so changing `lapse` widens the diff beyond its scope.
- Lens code-tests, `CheckAccept` test asserts the scope record alone: `CheckAccept` only reads, and an assertion on the accepting and kept directories would test a helper's absence of writes rather than a behavior of the public surface.
- Lens code-correctness, `t1.txt.bak` as a standalone finding: folded into F1, which fixes both match shapes.
- Lens code-tests, degenerate-name rows as a standalone finding: folded into F1's fix, which adds those rows.
- Lens code-correctness, `AcceptingDir` left holding partial files after a failed capture or a refused accept: the directory is ephemeral, the next capture empties it, and `Accept` reads it only after its own capture, so no state is wrong.
