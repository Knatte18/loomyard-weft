---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: PATTERN.md:49
    summary: PATTERN-recipe-setting-lookup lands without the word told, so it differs from the declared text and no Pattern-Departure trailer records the difference
    fix: In the PATTERN-recipe-setting-lookup line of PATTERN.md, change "via the owner's executable-key lookup" to "via the owner's told executable-key lookup", the declared text. The line then exceeds internal/pattern's 320-character MaxEntryLineChars, so shorten other wording on that same line (for example "Giving a recipe row a config value" to "Giving a recipe row a value"), or cut further outside the three declared entries, until the entry gate passes. If the declared text cannot fit at all, record the departure in a Pattern-Departure trailer on the commit that lands the line, naming PATTERN-recipe-setting-lookup.
    origin: lens:code-pattern-fit
  - id: F2
    severity: MEDIUM
    class: design
    location: internal/fabricengine/pendingflags.go:209
    summary: A holder's respawn of a pending code flag builds a request with no weft path, and the child it starts exits on the weft-path guard in coalescePushSides, so that code commit stays unpushed
    fix: In foreignFlagRequest, for the code side also set req.WeftPath to the records worktree of lockDir (filepath.Dir(lockDir), the worktree RecordsLockDirPath maps to lockDir), so the respawned child owes records too and passes coalescePushSides' weft-path requirement. Add a respawn row to TestCoalesceFlagged_PendingFlags that runs the built request through CoalescePushRequest for a code-side flag and expects the code commit pushed.
    origin: lens:code-correctness
  - id: F3
    severity: LOW
    class: design
    location: internal/fabricengine/spawn.go:160
    summary: statusPushRequest returns a warp-only request when only the records branch is marked local, and SpawnStatusPush still spawns a child that fails on the missing weft path
    fix: In statusPushRequest, when the records side is dropped as local-only while the code side stays, drop the code side from the request too and name its branch in LocalSkipped only if it is itself marked; or make SpawnStatusPush spawn nothing when WeftPath is empty. Pin it with a TestSpawnStatusPush row for a pair whose records branch alone is marked by hand.
    origin: lens:code-correctness
  - id: F4
    severity: LOW
    class: design
    location: internal/fabricengine/add.go:466
    summary: When the second ClearBranchLocalMark fails after the first succeeded, rollbackAdd keeps the adopted warp branch but it is already unmarked, so a later push could publish a pair meant to stay local
    fix: In add's final adoption loop, on a clear failure re-mark each branch already cleared (MarkBranchLocal on its repo) before calling rollbackAdd, and correct the comment above the loop that says a rollback never has a mark to restore. Alternatively clear both marks only after both reads succeed, so a failure leaves both set.
    origin: lens:code-correctness
  - id: F5
    severity: LOW
    class: decision
    location: internal/fabricengine/pushanchored.go:101
    summary: PushAnchored has no production caller left after the status seams and batten cleanup moved to spawning; only its integration tests use it
    fix: Decide whether PushAnchored stays as supported fabric API. If not, delete PushAnchored with its TestPushAnchored_* tests and fold its coverage into PushPairAnchored's rows. If it stays, say in its doc comment which caller class it is kept for.
    origin: lens:code-affected-set
  - id: F6
    severity: LOW
    class: consistency
    location: internal/fabricengine/spawn.go:10
    summary: The spawn.go file header, the PushWarpAt doc and the CoalescePushBothAt doc still say the detached child drives CoalescePushBothAt, but it now runs DetachedPush and CoalescePushRequest
    fix: In spawn.go update the header lines and the PushWarpAt doc paragraph that name CoalescePushBothAt as the child's push so they name DetachedPush, which runs CoalescePushRequest. In coalesce.go drop the CoalescePushBothAt doc sentence "the detached push child always supplies the weft path" and say its callers are foreground ones.
    origin: lens:code-affected-set
  - id: F7
    severity: NIT
    class: consistency
    location: internal/battencli/lifecycle_integration_test.go:474
    summary: A step comment still calls the Open-Window, Seed-IDE and Open-IDE rows the three switched rows
    fix: In the doc comment of stepFourRowRun_SeedsChildCommitsAndTearsDown replace "the three switched rows" with "the three Command rows".
    origin: lens:code-affected-set
---

### [BLOCKING] PATTERN-recipe-setting-lookup differs from its declared text

**Location:** PATTERN.md:49
**Issue:** The directive lists the entry's new text as "...via the owner's told executable-key lookup...". `PATTERN.md` carries it without "told". Plan card 15 chooses the omission to stay under the 320-character line limit, but the constraints bind over the plan, and no commit carries a `Pattern-Departure:` trailer. The other two declared entries, `PATTERN-push-both-sides` and `PATTERN-command-row`, land verbatim, and `pattern/PATTERN-push-both-sides.md` carries the matching bullet.

### [MEDIUM] Respawned code-side flag starts a child that cannot push

**Location:** internal/fabricengine/pendingflags.go:209
**Issue:** `foreignFlagRequest` keeps only `WarpPath` for a code flag. `DetachedPush` hands it to `CoalescePushRequest`, and `coalescePushSides` returns an error when `WeftPath` is empty. The child only logs the failure, so the commit that raised the flag stays unpushed, which the protocol's no-lost-push guarantee rules out. It is reachable when a holder that owes only the records side finds `pending.code`, for example after a code side was dropped as local-only. Card 8 describes this respawn as working.

### [LOW] statusPushRequest can return a warp-only request

**Location:** internal/fabricengine/spawn.go:160
**Issue:** With only the records branch marked local, the request keeps the warp path and drops the weft path. The spawned child fails on the same weft-path guard as the finding above. It needs a hand-marked mixed pair, so it is rare.

### [LOW] Second mark clear can fail after the first succeeded

**Location:** internal/fabricengine/add.go:466
**Issue:** The comment says no failable step follows, so a rollback never has a mark to restore. If the weft clear fails after the warp clear succeeded, `rollbackAdd` leaves the adopted warp branch in place but unmarked, so a later fabric push could publish a pair that was meant to stay local.

### [LOW] PushAnchored is exported with no production caller

**Location:** internal/fabricengine/pushanchored.go:101
**Issue:** After card 10 the status seams and batten cleanup spawn the push, and `loomcli` uses `PushPairAnchored` for commit-records. The function and its protocol rewiring are kept alive only by integration tests, and no card deletes it.

### [LOW] Stale comments naming the old child entry point

**Location:** internal/fabricengine/spawn.go:10
**Issue:** The spawn.go header, the `PushWarpAt` doc and the `CoalescePushBothAt` doc describe a child that drives `CoalescePushBothAt`. The diff changed the child's body in `weft_verbs.go`, and only foreground callers such as `hubforge` still call `CoalescePushBothAt`. `doc.go` was updated; these were not.

### [NIT] Switched-rows wording left in a test comment

**Location:** internal/battencli/lifecycle_integration_test.go:474
**Issue:** The rows are `Command` rows after card 15.

## Rejected

- Fork code-tests: `PushPairAnchored` has no local-skip test. `pushanchored_integration_test.go:437` asserts `LocalSkipped` for a marked warp branch.
- Fork code-tests: `unpushedOutcome`'s `ErrLocalBranch` case is unpinned. `coalesce_integration_test.go:90` covers a marked warp branch in the coalescing push.
- Fork code-pattern-fit: `git branch -D` in the `ErrLocalAddOverOrdinary` text breaks `PATTERN-no-denied-recovery`. The rule covers webster, shed and loom refusals, stencils and specs, and this is a `fabric add` message to an operator.
- Fork code-pattern-fit: `shedadapters` importing `verifyrun` may break `PATTERN-leaf-packages`. The background file names neither package.
- Fork code-pattern-fit: about fifty undeclared `PATTERN.md` rewordings. Plan card 10 mandates and lists this cut to stay inside the line-length and headroom budget, and I found no entry outside the listed ones.
- Handler: no test forks a real push child, and no status seam still pushes inline except `commit-records`, which the declared `PATTERN-push-both-sides` text keeps synchronous. `SpawnDetachedPush` returns `ErrSpawnUnderTest` in a test binary.
- Handler: no leftover caller of the removed switched rows or engines. The commands are read through `configreg.SettingLookup` over the board layer only, and the three command keys are `CategoryExecutable`.
