---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/shuttleengine/claudeengine/resume.go:302
    origin: lens:code-correctness
    summary: "A known config directory with no sessions directory reads as an unreadable registry, so checkResumable warns and relaunchSpec refuses; the first switch onto an account that never ran a session ends the run died, and processLiveness can never read dead."
    fix: "In checkResumable, sessionHolder and processLiveness, treat a registry directory that does not exist (errors.Is(err, fs.ErrNotExist)) as an empty registry, and keep every other read failure unreadable. Correct the rows TestProcessLiveness secondNoDir, TestSessionHolder secondMissing and the dead-entry-beside-an-unreadable-second-registry row, which pin the opposite, and add rows where a live entry in one directory plus a missing second directory reads alive, and a dead entry plus a missing directory reads dead."
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/shuttleengine/accountrelaunch_test.go:187
    origin: lens:code-tests
    summary: "No test pins D7 step 4's ordering invariant: the record names the new session id, PromptLine, cleared started flag and offsets before reed relaunches the strand. relaunchReed.RelaunchStrand never reads the record, and the rows read it only after Wait returns."
    fix: "In TestWait_UsageLimitSwitchRelaunchesInTheRun, have the relaunchReed onRelaunch hook (or RelaunchStrand) load the run record with loadRunState and fail the test unless it already holds the relaunched session id, the empty PromptLine, Started false, RelaunchingAt set and PromptOffset and LaunchOffset equal to the events file size. Add the same read to a start-path row."
  - id: F3
    severity: LOW
    class: design
    location: internal/shuttleengine/stop.go:231
    origin: lens:code-correctness
    summary: "When EndStrandProcess fails over a live strand and ReleaseStrandHold then fails too, the failure is only logged; the strand stays held, so reed neither relaunches nor drops it if it later dies, and nothing clears the hold."
    fix: "In stopForRelaunch, return the release failure joined onto the returned error, and make the Wait died path (endFailedRelaunch and settleStaleRelaunch) remove a strand reed reports held whatever the mark says. State in reedengine/doc.go who owns clearing a hold."
  - id: F4
    severity: LOW
    class: design
    location: internal/shuttleengine/apierrorretry.go:279
    origin: lens:code-correctness
    summary: "apiErrorWatch.turnEnds restarts at 0 for each Wait or attach while the cursor starts at PromptOffset, which lies past any load turn end; for a copied resume with a load turn the first task-turn API error counts as the first turn end and is read as a refused resume, discarding a session the load turn proved resumed."
    fix: "Count the first turn end from LaunchOffset rather than from the cursor: seed watch.turnEnds in retryAPIError from the turn ends between run.state.LaunchOffset and the cursor (or persist that the first turn end was seen by advancing LaunchOffset past it), and add a row to TestWait_RefusedResumeStartsFresh with a copied resume whose load turn ended and whose next turn end is an API error."
  - id: F5
    severity: LOW
    class: design
    location: internal/shuttleengine/claudeengine/accountconfig.go:131
    origin: lens:code-correctness
    summary: "validateAccounts expands a leading ~/ with filepath.Join(home, ...) even when the home directory is empty, so ~/x becomes /x, which passes the absolute-path check; LoadConfig's comment says the refusal happens."
    fix: "In validateAccounts, when home is empty and config_dir is ~ or starts with ~/, return refuse(...) naming the key, the config_dir and that the home directory is unresolved, before the Join. Add a row to TestLoadConfig."
  - id: F6
    severity: LOW
    class: consistency
    location: internal/reedcli/resume.go:25
    origin: lens:code-affected-set
    summary: "The lym reed resume help text says held names only the strands a covering pause left down, but ResumeResult.Held now also lists strands a caller ended on purpose (Strand.Held)."
    fix: "Update the help text and any description of the held envelope field in internal/reedcli/resume.go to name both causes, matching reedengine/doc.go."
  - id: F7
    severity: LOW
    class: scope
    location: internal/testcover/mutantbuild_integration_test.go:120
    origin: lens:code-scope
    summary: "Commit c3f46285 clips a shared env slice in a testcover fixture; it traces to no decision or plan card and no traced change forces it."
    fix: "Drop the testcover hunk from this branch and land it separately; if the verify gate of this branch needs it, leave it and have the parent record the reason in the landing summary."
  - id: F8
    severity: LOW
    class: consistency
    location: internal/orchengine/restart_test.go:230
    origin: lens:code-tests
    summary: "Three tests the redundancy report lists as contained each pin an assertion their covering tests never make, so each needs a testtiming:keep marker instead of staying unmarked."
    fix: "Add a line //testtiming:keep pins the fresh restart and the refused-resume-only ask, which the usage-limit test never drives directly above func TestWatcher_RefusedResumeStartsTheOrchFresh in internal/orchengine/restart_test.go. Add //testtiming:keep pins the verdictError of a relaunching mark over an untracked strand, which no covering test reaches above func TestDispositionCandidate_RelaunchingRecord in internal/shuttleengine/attach_asking_test.go. Add //testtiming:keep pins that NewFromConfig tells no worktree, which no covering test asserts above func TestNewForWorktree in internal/shuttleengine/claudeengine/worktree_test.go."
  - id: F9
    severity: LOW
    class: scope
    location: internal/shuttleengine/stepstop.go:78
    origin: lens:code-tests
    summary: "StopOrphanedStepRuns now judges liveness with runIsLive over now and the bound, and StopStepRuns stops a relaunching run past its bound; no test pins a relaunching run within its bound being left alone by the orphan stop, or the past-bound stop."
    fix: "Add a relaunching-within-bound row (record mark one second old, dead pid) to the StopOrphanedStepRuns test expecting the run is not listed, and a mark-past-bound row to TestRunner_StopStepRuns_StopsOnlyTheStepsRuns expecting it stopped and its held strand removed."
  - id: F10
    severity: LOW
    class: scope
    location: internal/shuttleengine/accountrelaunch.go:258
    origin: lens:code-tests
    summary: "The declined-relaunch path (errRelaunchDeclined when the mark write fails), the switch-error branches of switchAccountAtLimit and switchAtLoadLimit, and the orch watcher's failing Fresh restart and failing RefusedResume ask have no test row."
    fix: "Add rows that make the record write fail before the mark and assert the limit is handled as without a switch with handledOffset restored; a row where SwitchAccount returns an error and the run is left as it was; in internal/orchengine/restart_test.go a row where the Fresh restart errors and the watcher ends naming lym beam start, and a row where RefusedResume errors and nothing restarts."
  - id: F11
    severity: LOW
    class: scope
    location: internal/shuttleengine/accountrelaunch.go:341
    origin: lens:code-tests
    summary: "relaunchRepeat over a spec that resumes a session has no test, though the parent decision requires the same session id to be resumed with its transcript carried when a load turn hits a usage limit; the no-transcript fallback of the start-path repeat is also unpinned."
    fix: "Add a row to TestStart_UsageLimitedLoadTurnRepeatsTheLaunch with resumes set and the usage-limit turn end, asserting the second Prepare spec keeps ResumeSessionID and the relaunch record keeps TranscriptCopied, plus a row where CheckResume returns ErrNoTranscript and the repeat starts fresh."
  - id: F12
    severity: LOW
    class: scope
    location: internal/shuttleengine/claudeengine/accountkeeper.go:593
    origin: lens:code-tests
    summary: "No test fails if PickAccount reads the hub's worktree names outside the account file's lock; the rows use a fixed names list, so D2's rule that a worktree made meanwhile keeps its entry is unpinned."
    fix: "In TestClaude_PickAccount add a row whose Names func, when called, writes an entry for another worktree into the account file through updateAccountFile from a second goroutine and asserts the call blocks until the pick returns or, simpler, asserts the account file lock is held while Names runs by trying a non-blocking write lock on accountFilePath inside Names."
  - id: F13
    severity: LOW
    class: scope
    location: internal/shuttleengine/claudeengine/transcriptread.go:516
    origin: lens:code-tests
    summary: "The env module's CLAUDE_CONFIG_DIR as a known directory for LocateSessions and the registries is untested, and so is the sessionProjectDir tier that reads the newest known copy when neither a live entry nor the resolved directory holds the transcript."
    fix: "Add an engine row to TestLocateSessions with an env module vars CLAUDE_CONFIG_DIR directory holding a session, and a row to TestClaude_LiveReadersReadTheLaunchedDirectory with no live entry, the resolved directory lacking the transcript and another known directory holding it."
---

### [MEDIUM] A missing sessions directory breaks the multi-directory registry readers

**Location:** internal/shuttleengine/claudeengine/resume.go:302
**Issue:** `readRegistry` returns the `ReadDir` error for a directory that does not exist. D8 reads every known directory as one set, and that set always holds `~/.claude`, retired accounts and listed accounts that never ran. `checkResumable` turns the failure into a warning, and `relaunchSpec` treats any warning as a refusal, so the first switch onto such an account ends the run as died. `processLiveness` counts the missing directory as doubt and never reads dead. The existing rows pin that behaviour, so they pin the defect.

### [MEDIUM] D7 step 4's record-before-relaunch ordering has no test

**Location:** internal/shuttleengine/accountrelaunch_test.go:187
**Issue:** The decision's central safety property, and an acceptance criterion, is that the run record names the new session, its `PromptLine`, the cleared started flag and the offsets before reed relaunches the strand. No row reads the record inside the relaunch. Moving the record write after `RelaunchStrand` would pass every row.

### [LOW] A failed release leaves a live strand held

**Location:** internal/shuttleengine/stop.go:231
**Issue:** After a failed `EndStrandProcess` over a live strand, `ReleaseStrandHold` is tried once and its failure only logged. The held mark survives, so reed's resume skips the strand if it later dies, and nothing but a stop or the died path clears it.

### [LOW] Refused-resume detection counts turn ends from the attach cursor

**Location:** internal/shuttleengine/apierrorretry.go:279
**Issue:** `turnEnds` restarts at 0 per `Wait` and the cursor starts at the prompt offset. A copied resume that took a load turn then reads its first task-turn API error as the resume's first turn end and restarts fresh, discarding a session the load turn showed was resumable.

### [LOW] An unresolvable home accepts a ~/ config directory

**Location:** internal/shuttleengine/claudeengine/accountconfig.go:131
**Issue:** With an empty home, `filepath.Join("", "/x")` yields an absolute path, so the refusal `LoadConfig` promises for `~/` never fires and the account points at the filesystem root.

### [LOW] The reed resume help text is stale

**Location:** internal/reedcli/resume.go:25
**Issue:** The help text says `held` names only strands a pause left down. `ResumeResult.Held` now also lists strands a caller ended on purpose.

### [LOW] An unrelated testcover change rides on this branch

**Location:** internal/testcover/mutantbuild_integration_test.go:120
**Issue:** Commit `c3f46285` fixes a parallel-subtest environment slice in `testcover`. It traces to no decision or card and is not forced by a traced change.

### [LOW] Three contained tests lack the keep marker their unique assertions earn

**Location:** internal/orchengine/restart_test.go:230
**Issue:** `TestWatcher_RefusedResumeStartsTheOrchFresh` drives the `RefusedResume` ask and the `Fresh` restart, which `TestWatcher_UsageLimitSwitchRestartsTheOrch` never asserts. `TestDispositionCandidate_RelaunchingRecord` pins the `verdictError` of an untracked strand, which none of its covering tests reaches. `TestNewForWorktree` pins that `NewFromConfig` keeps the zero worktree, which no covering test asserts. Each pins what coverage cannot see, so each is kept, not deleted.

### [LOW] Orphan and step stops of a relaunching run are only partly pinned

**Location:** internal/shuttleengine/stepstop.go:78
**Issue:** The `runIsLive` change in `StopOrphanedStepRuns` and the past-bound arm of `StopStepRuns` have no row. Only a one-second-old mark is tested, and only through `StopStepRuns`.

### [LOW] Failure branches of the relaunch and the orch restart are untested

**Location:** internal/shuttleengine/accountrelaunch.go:258
**Issue:** The declined relaunch, which restores `handledOffset`, the switch-error branches, and the orch watcher's failing `Fresh` restart and `RefusedResume` error have no row.

### [LOW] A repeated launch of a resumed session is untested

**Location:** internal/shuttleengine/accountrelaunch.go:341
**Issue:** The parent decision makes the repeat resume the same session id with its transcript carried. The only resumed start row uses the fresh relaunch, and the no-transcript fallback of the start-path repeat is unpinned.

### [LOW] The pick's names read under the lock is unpinned

**Location:** internal/shuttleengine/claudeengine/accountkeeper.go:593
**Issue:** D2 says the pick calls the names reader under the file's lock so a worktree made meanwhile keeps its entry. The rows use a fixed list, so moving the call outside the lock passes them.

### [LOW] Two known-directory sources are untested

**Location:** internal/shuttleengine/claudeengine/transcriptread.go:516
**Issue:** The env module's directory as a known directory, and the `sessionProjectDir` tier that reads the newest known copy, have no row.

## Rejected

- code-tests: `SetAccount` on an engine told no worktree has no row. The guard is a one-line refusal, and a row would assert the error text alone.
- code-tests: `SessionNameDrift` and `SessionUsage` have no launched-directory test of their own. Both call `sessionProjectDir`, which `TestClaude_LiveReadersReadTheLaunchedDirectory` already pins for the other readers.
- code-tests: no test for "reconcile leaves a held record", and none for `EndStrandProcess` failing after the mark was saved. The first is a documentation claim about code the diff does not change; the second is covered by the mark-before-pane ordering the tier-2 test exercises.
- Handler check: D5's "also with no creel flag directory" for `OpenLaunch` is met. `creelFlagDirForHook` is empty only when the engine has no board directory, and an engine with none lists no account, so the case cannot arise.
- Handler check: the declared `PATTERN-shuttle-provider-seam` change lands in `pattern/PATTERN-shuttle-provider-seam.md` with the declared text, and no commit of the range carries a `Pattern-Departure:` or `Loosened-Check:` trailer.
