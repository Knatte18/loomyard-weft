---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/battencli/dobbyreads.go:172
    summary: An unreadable landed-commit file aborts the whole DobbyRuns listing instead of yielding landed, commit unknown
    fix: In the landed-commit read closure of DobbyRuns, when shedrun.ReadLandedCommit returns an error, log it with logger.Warn (slug and error) and leave view.LandingCommit empty, returning nil from the closure, so the run lists as landed with an unknown commit and the other runs are driven. Add a row to TestDobbyRuns_ReadsTheLandOutcomeAndTheLandingCommit where the landed-commit path is a directory and the run still lists with an empty LandingCommit.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/landingshed/finalize_test.go:457
    summary: The recorded landing commit on a first-try parent-side merge is asserted by no test, so removing that recordLanding call passes
    fix: Add a row to TestFinalize_MergeInRequiredRetry, or a first-try case beside TestFinalize_RecordLandingFailureKeepsDone, whose RecordLanding seam appends the commit to the order and asserts the order ends mark, record landed1, push for a merge that lands at the first call.
  - id: F3
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/dobbyengine/landnotice_test.go
    summary: The Requested refresh while the entry is on the board and the kept reading on a failed board read have no row
    fix: Add rows to TestLandNotices that tick twice before Land with the entry's notify_on_land flipped between ticks and assert the line follows the later value under per-task, and one where the Tasks seam fails on the second tick and the first reading still decides.
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/boardengine/board_test.go:744
    summary: The revised date and board replay ignoring or carrying notify_on_land are asserted by no test
    fix: In TestNotifyOnLand add an assertion that setting the field on an entry leaves its revised date unchanged, and add a TestFabricReplay row where the committed side sets notify_on_land and the replayed entry carries it.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/loomcli/landingdeps.go:199
    summary: The loomcli RecordLanding wiring and the beamcli ClearNotifyOnLand wiring are pinned by no test, so the write path and the read path of the landed-commit file could drift apart
    fix: Add a test in loomcli that runs the landingDeps RecordLanding seam for a hub with a prime and asserts shedrun.ReadLandedCommit(prime, worktreeName) returns the commit, and a test in beamcli that runs the hubDobbyDeps ClearNotifyOnLand seam against a hub board and asserts the field is cleared.
---

### [MEDIUM] An unreadable landed-commit file aborts the run listing

**Location:** `internal/battencli/dobbyreads.go:172`
**Issue:** The landed-commit read inside `DobbyRuns` returns `fmt.Errorf("read the run %s: %w", ...)` for any error but a missing file, and `DobbyRuns` returns `nil, err` on it. A done run stays in the listing, so the read repeats every tick; `drivePasses` then fails with "list the runs", and one bad file (a directory, a permission error, an I/O error) stops every run's passes for good. D6 says a failed record never changes a verdict and the focus asks that a missing or unreadable file yield `landed, commit unknown`; only the missing file does. I checked the closure and the error loop at `dobbyreads.go:166-179` and `ReadLandedCommit` in `shedrun/landed.go`.

### [MEDIUM] The first-try record of the landing commit is unasserted

**Location:** `internal/landingshed/finalize_test.go:457`
**Issue:** `recordLanding` is called at two sites in `Finalize.Call`, after the first parent-side merge and after the merge-in retry. `TestFinalize_MergeInRequiredRetry` asserts the recorded commit and its order only on the retry site. `TestFinalize_RecordLandingFailureKeepsDone` takes the first-try path but its seam discards the commit and asserts only the push. Deleting the first-try call, the main landing path, fails no test, yet the acceptance criterion is that Finalize records the parent's head after a successful parent-side merge.

### [LOW] The Requested refresh and the failed board read are unpinned

**Location:** `internal/dobbyengine/landnotice_test.go`
**Issue:** `TestLandNotices` fixes `Requested` to one value per row. The refresh while the entry is on the board, and the rule that a failed board read keeps each remembered request, are behaviors of `landNotices` no row would fail on. The pruned-before-Land case, the one the focus names, is covered by the existing row that keeps the request after the prune.

### [LOW] The revised date and replay behaviors of the field are unasserted

**Location:** `internal/boardengine/board_test.go:744`
**Issue:** D3 says the revised date ignores the field and replay carries it. `TestNotifyOnLand` and the `TestRunLock` row check acceptance and storage, not the date, and `TestFabricReplay` has no row with the field, so removing `replayFields`' new row or letting the field move the date fails no test.

### [LOW] The two deps wirings of the landing notice are unpinned

**Location:** `internal/loomcli/landingdeps.go:199`
**Issue:** The write side (`loomcli` `RecordLanding`, through `primeOfHub` and `seedSlug(l.WorktreeName)`) and the read side (`battencli.DobbyRuns`) name the same file in the code I read, but no test ties them. `beamcli`'s `ClearNotifyOnLand` closure has no test either. Both are thin closures, so the severity is LOW.

## Rejected

- Settling the post but failing the memory drop re-posts the line (code-correctness 2): it needs the dobby's own state file to fail to write, and then `tick`'s own `UpdateState` fails the tick as well; not worth a mechanism.
- A transiently unreadable run status drops its notice memory (code-correctness 3): it follows D4's "no longer listed" rule as written, and the decision record lists the loss of a line on a dobby gap under its bounds.
- The upsert help text rewrites four existing field descriptions (code-affected-set 1): the 2000-byte cap forces the trim, and the change description discloses it.
- Long help-text lines in `overview.go` and `beamcli/cli.go` (code-affected-set 2): line breaks of comments and help are never a finding by the rubric.
- No row for a `LoadConfig` failure, a failed clear, a nil `ClearNotifyOnLand`, a short commit, the `tick` call site, and the other upsert paths (code-tests 3, 5, 6, 7): each is a one-branch case whose code I ran by hand and found correct; the tick order is pinned by every `h.tick` row that lists a done run.
- The overview header tests at `overviewdobby_test.go:111` (code-affected-set, unverified): the verify gate decides a stale assertion.

I settled the focus questions this way: a run pruned before Land keeps its last `Requested` and still sends under `per-task`; the closure is retried safely since `due` resets on entry; a failed post is retried on the next tick up to three attempts and the done run still reaches the step because `listedRuns` is cloned before done runs drop; a `LoadConfig` failure leaves the memory until the next good load and `off` clears it; the call site in `tick` follows D5; `recordLanding` runs after each successful parent-side merge, before the push, and a failure only logs; the `PastLand` extraction leaves no copy of the rule; the board field, the config key and the overview match D2, D3 and D7. `internal/testkit/envkit/envkit.go` gains only the `nilLegal` entry for the new nil-legal seam, which the new seam needs.
