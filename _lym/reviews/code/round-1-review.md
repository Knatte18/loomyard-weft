---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/boardengine/raddlefile.go:81
    summary: A failure or crash between FileRaddle's board save and its store write leaves filed entries with no done marker, and the retry sends the proposal back as slug-taken until the returns are spent
    fix: >-
      Make step 3 resumable from a half-done write.
      In internal/boardengine/raddlefile.go FileRaddle, after the done-marker check and before f.Check,
      test whether the board already holds every slug of f.ProjectEntry (when set) and f.Parts as a discussed entry whose body begins "Filed from the Sketch `<f.Sketch>`" (parts) or "Groups the entries filed from the Sketch `<f.Sketch>`" (project).
      When it does, skip f.Check and fileRaddleEntry and return raddleFiled{released: the part slugs when f.Release is set},
      so the write only runs afterSave, which writes the store directory with store.json last.
      In internal/loomshed/raddlefile.go add a RaddleFileDeps seam AlreadyFiled(sketch string, slugs []string) (bool, error), non-nil-checked in internal/shedrecipe/entries_raddle.go raddleFileEntry and wired in internal/loomcli/raddlewiring.go raddleFileDeps by the same body test over the hub board.
      In raddleFile.Call skip verify when the store is not done but AlreadyFiled is true, since RecheckBoard reports the run's own filed slugs as taken (internal/raddleparser/recheck.go:38), and still run stageAndFile so the filing and the store are written.
      Add a TestFileRaddle row in internal/boardengine/raddlefile_test.go that fails the store write after the save and then calls FileRaddle again, and a loomshed raddlefile_test.go row where StoreDone is false and AlreadyFiled is true.
  - id: F2
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: docs/overview.md:142
    summary: The overview still describes _lym/raddle/ as a raddle nav-doc overlay and has no row for _lym/usage/, though this diff makes the first a raddle run's directory and adds the second
    fix: >-
      In docs/overview.md replace the `_lym/raddle/` row at line 142 with a description of what a raddle run keeps there: its proposal, cold proposal, approval record, state, staged store files and part records, reached through the `_lym` junction.
      Add a row for `_lym/usage/` beside it, naming the per-session usage files, run.json and landed.json that a run keeps (internal/lymdirs/dirs.go UsageDirName), with the same Records worktree and Records columns as the other durable `_lym` subtrees.
      Rewrite the sentences at lines 188-189, which say raddle content is anchor-level, has no `_raddle` junction and points at the board's `raddle` note, to say the same about the raddle run's directory without the pointer to the note.
  - id: F3
    severity: MEDIUM
    class: scope
    origin: lens:code-tests
    location: internal/boardengine/raddlefile_test.go:90
    summary: FileRaddle's done-marker branch and loomcli's real StoreDone closure have no test, though step 3's skip after a retry rests on them
    fix: >-
      In internal/boardengine/raddlefile_test.go add a TestFileRaddle row that files a Sketch, then calls FileRaddle again with the same Sketch, created stamp and hash and asserts the second call writes nothing: board.json, the entries and the store directory are unchanged and no error is returned.
      Add a row where the stored hash differs (or the created stamp differs) and assert the call does not take the marker branch.
      In internal/loomcli/raddlewiring_test.go add a table test of the StoreDone closure from raddleFileDeps over a temp board directory holding a store.json: true for the matching created stamp and hash, false for another hash, another created stamp, another Sketch slug and an empty store.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/loomcli/raddleverbs.go:115
    summary: raddle-propose records the session as proposed before it posts the waiting notice, so a failed notice lets raddle-approve pass in a session that sent none
    fix: >-
      In internal/loomcli/raddleverbs.go raddleApproveVerb refuse, beside the ProposedSession check at line 153, when stored.NoticeSession < stored.Session,
      with a message naming the way forward: run lym loom raddle-propose again, which posts the waiting notice, then raddle-approve on the operator's approval.
      Add a row to the raddleverbs_test.go approve table whose state holds ProposedSession equal to Session and NoticeSession below it, and update any existing approve row that seeds ProposedSession without NoticeSession.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/loomshed/recordwrite.go:136
    summary: After a crash that wrote the stored record but not the support log, the resume writes the board-rendered support log beside the stored record
    fix: >-
      In internal/loomshed/recordwrite.go Call, choose supportLog from whether the entry has a stored record whenever the log is missing, not only inside the recordMissing branch:
      when logMissing, call p.deps.ReadStored and use StoredRecordSupportLog() when it reports found, boardSupportLog() otherwise.
      Add a row to recordwrite_test.go where the record file exists, the support log is missing and ReadStored finds a record, asserting the written log is StoredRecordSupportLog().
---

### [MEDIUM] A half-done filing is never finished by a retry

**Location:** `internal/boardengine/raddlefile.go:81`, with `internal/boardengine/board.go:176-181` and `internal/raddleparser/recheck.go:38-41`
**Issue:** `boardCriticalSection` saves `board.json` and only then runs `afterSave`, and `FileRaddle` writes the raddle store directory, `store.json` last, in `afterSave`. A crash, or a failed store write, between the two leaves the filed entries on the board and no done marker. On the retry `StoreDone` reads false, so `verify` runs, and `RecheckBoard` reports each filed slug as taken because only the Sketch's and the run's own slugs are exempt. The proposal goes back to Raddle-Write, each send-back spends one of the two returns, and the third failure halts the run with the entries filed and unrecorded. D9 asks that a retry after a crash finish the steps it missed and repeat none, and D10 relies on the store holding every filed slug.

### [MEDIUM] The overview describes the raddle directory and usage directory as before the change

**Location:** `docs/overview.md:142`, `docs/overview.md:188`
**Issue:** Line 142 still calls `_lym/raddle/` "Raddle documentation (the raddle nav-doc overlay)", and lines 188-189 still say it has no `_raddle` junction and point at the board's `raddle` note. This diff makes `_lym/raddle/` the directory of a raddle run's proposals, approval, state, staged store and part records (`internal/lymdirs/dirs.go:52`). The `_lym/` artifact table also has no row for `_lym/usage/` (`internal/lymdirs/dirs.go:50`), although it lists the other durable subtrees. The diff edits other parts of this file but leaves these.

### [MEDIUM] Step 3's done marker is untested where it is decided

**Location:** `internal/boardengine/raddlefile_test.go:90`
**Issue:** D9's acceptance criterion says a Raddle-File interrupted after step 3 finds step 3 done by the store's hash, also after a part has landed and been pruned. The `loomshed` retry tests drive a fake `StoreDone`, and the `TestFileRaddle` subtests are the shape table, the re-check refusal, `TestReleaseTasks` and `TestEndRaddleEntry`. No row repeats `FileRaddle` over a held store, and the real `StoreDone` closure in `internal/loomcli/raddlewiring.go:180` has no test, so a change to the created-stamp or hash comparison would break no row.

### [LOW] The waiting notice is not required by raddle-approve

**Location:** `internal/loomcli/raddleverbs.go:115`
**Issue:** `raddle-propose` sets `ProposedSession` first and posts the notice after it. When the notice fails the verb returns an error with the session already marked proposed, and `raddle-approve` checks only `ProposedSession`. D7's Bound says the waiting notice goes out in every session that approves, which this path breaks until the agent happens to run `raddle-propose` again.

### [LOW] A resumed Record-Write may write the wrong support log

**Location:** `internal/loomshed/recordwrite.go:136`
**Issue:** `supportLog` defaults to the board-rendered log and becomes the stored-record log only inside the branch that writes a missing record. After a crash between the record write and the log write, the resume finds the record present and the log missing, and writes the board-rendered log beside a stored record. The two files then describe different origins.

## Rejected

- Lens code-tests, no test that a raddle run's finish never makes its entry a Sketch: `finishesAsSketch` is tested for a recipe with a `RaddleFile` row in `internal/loomcli/recipekeyed_test.go:70`, which this diff edits.
- Lens code-tests, no test that every run runner installs the usage recorder: the runner's `sessionEnd` field is unexported and observable only through a spawned session, the hook call is pinned in `internal/shuttleengine/sessionend_test.go` and the writer in `internal/shedadapters/sessionusage_test.go`, and card 5 prescribes only `TestUsageRecorder`; a wiring test would be a heavy fixture for a one-line call at each site.
- Lens code-tests, no row for a Finalize with no kept parent head: `internal/landingshed/finalize_test.go:523-536` covers "not at all when no merged parent head is held".
- Lens code-tests, `lym report cost` and `lym report raddle` output paths pinned at engine level only: the lens did not check the CLI layer and the engine tests pin the report content; the CLI passes the engine's result through, so a test there would repeat the engine's.
- Lens code-tests, `lym board promote --recipe raddle` through the CLI: the recipe flag is shared with the darn row's pass-through to `PromoteTo`, which `TestDiscussedPromotion` pins at engine level.
- Handler, runners in `burlercli`, `webstercli`, `orchcli`, `creelcli`, `tendercli`, `pairteardown` and `shuttlecli` carry no usage hook: card 5 names the run's runner sites and the stops it leaves hookless (`_lym/plan/05-usage-wiring.md`), and each named site carries the hook.
