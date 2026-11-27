---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/pairteardown/leftover.go#RemoveBenchPairs
    summary: A bench pair whose own run lock is held, such as by a loom or shed driver still running in it, is refused rather than ended, so the task's Worktree-Teardown row goes Stuck on every resume until that bench run stops by itself; the doc comment, pairteardown/doc.go and the change description all claim the teardown ends a live process instead of refusing it.
    fix: "In RemoveBenchPairs, end the bench pair's registered processes before td.Run, as RemoveHandedOverPair does with EndSubjectProcesses(prime, slug, false), so that the run-lock holder is gone when EndSession's awaitOrPass waits for the run lock. Add a TestRemoveBenchPairs row that holds the bench pair's run lock from a live registered process and expects the pair removed."
  - id: F2
    severity: MEDIUM
    class: design
    origin: handler
    location: internal/battencli/wire.go:1245
    summary: RemoveBenchPairs runs inside the Remove dep, which the Worktree-Teardown producer calls under its second prime-lock hold, so the hub's prime lock is held across each bench pair's archive-tag pushes to origin and its full teardown, including run-lock waits; the producer's design runs every push with no prime lock held so that no network wait holds the lock.
    fix: "Move the RemoveBenchPairs call and its log line out of the Teardown.Remove closure to the top of the Teardown.Push closure in internal/battencli/wire.go, which the producer runs with no prime lock held, returning its wrapped error from Push so that a failure still keeps the task pair and the resumed row retries. Update the Remove comment and the battenshed/doc.go sentence to say the push half removes the bench pairs."
  - id: F3
    severity: MEDIUM
    class: design
    origin: handler
    location: internal/fabriccli/roleguard.go#benchPairVerb
    summary: The guard admits add and remove of any <slug>-b<N> slug, including one that another live task's run owns, which breaks the answer key's bound that a bench never touches another task's pair; an add of a slug whose task has not started leaves a pair that blocks that task's Worktree-Create and that the teardown then leaves standing.
    fix: "In benchPairVerb, refuse when a batten run directory names the argument slug: resolve the hub's prime location from the task worktree's location and stat shedrun.RunDir(prime, slug), refusing on any result but fs.ErrNotExist. Add guard rows for add and remove of a slug whose run directory exists. A board-only clash stays undetectable under PATTERN-fabric-boundary; state that in the benchPairVerb doc comment and in the summary's design decisions."
  - id: F4
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/landing/summary.md
    summary: "The Merge danger section calls the door two-way with blast radius teardown alone, yet the diff pushes owner archive tags to origin, force-removes pairs and ends their live processes, and widens the sandboxed-role guard's admission of mutating fabric verbs; the blast radius omits the guard."
    fix: "In _lym/landing/summary.md's ## Merge danger, set the door to one-way, and widen the blast radius to name the fabric role guard's add, remove and push admission for darn writers, and the teardown's removal of bench pairs with archive tags pushed to origin."
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/battencli/wire.go:1242
    summary: No test pins the Worktree-Teardown wiring this diff adds, which removes the bench pairs before the task pair and halts the row with the task pair standing when a bench removal fails; a reordering or a dropped return on the error passes every test.
    fix: "Add a battencli test, at the lowest tier that drives the wired Teardown deps over a hubforge hub, that holds a task pair and a <slug>-b1 bench pair. The test asserts that the removal half removes both pairs. A second row makes the bench archive fail through a refusing pre-receive hook and asserts that the error names the bench pairs and the task pair still stands."
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/pairteardown/leftover.go#RemoveBenchPairs
    summary: A half-removed bench pair, either listed prunable or one whose worktree does not resolve, is skipped with no log line and the row reports success, so its records sibling and branches outlive the run with nothing recording that it was left.
    fix: "In RemoveBenchPairs, log at Warn each skipped worktree whose path base name is a bench pair slug of taskSlug, for the Prunable case and for the ResolveWorktree-error case, naming the slug and the reason and the way forward lym fabric prune --apply."
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/fabriccli/roleguard.go#benchPairVerb
    summary: A run identity record that cannot be read or decoded makes the guard refuse silently, and the refusal then appends the bench pair way forward, telling the writer to run the command it just ran; the read error is dropped.
    fix: "Make benchPairVerb return the ReadRunUsageRecord error alongside its bool. In sandboxedRoleRefusal, append that error to the refusal instead of benchPairWayForward when it is non-nil."
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/fabriccli/roleguard_test.go
    summary: Two refusals benchPairVerb makes have no row, an undecodable run.json and an invocation with more than one positional argument, so removing either check passes the tests.
    fix: "In the bench subtest of TestSandboxedRoleGuard, add a row for a task worktree whose _lym/usage/run.json holds invalid JSON, expected refused, and a row for add slug-b1 slug-b2, expected refused."
  - id: F9
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/pairteardown/leftover_integration_test.go#TestRemoveBenchPairs
    summary: The error row returns before it asserts the removed slugs, so the contract that the first failure returns with the slugs removed before it is unpinned.
    fix: "In TestRemoveBenchPairs, add a wantRemoved assertion to the error path, so that the refused-archive row checks removed against the slugs the run removed before the failing pair, with the order fixed by making only task-b1 hold a commit origin lacks."
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:code-scope
    location: internal/fabriccli/roleguard.go#isBenchPairPath
    summary: The push admission checks only that the strand is a darn writer and the slug has the bench form, never the run's recipe and docs profile that benchPairVerb checks, so it is wider than the bound the answer key sets on the new access.
    fix: "In isBenchPairPath, also require that the strand's own worktree for the bench pair's hub holds a run identity record naming shedrun.RecipeDarn and runprofile.DocsProfile, sharing the check with benchPairVerb through one helper. Add a push row for a writer on another profile, expected refused."
  - id: F11
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: pattern/PATTERN-pair-remote.md
    summary: The Pair teardown section lists the routes that skip the driver checks and the leftover teardown routes, but not the new bench pair route, which passes an ended verdict for a pair with no entry and ends its live processes.
    fix: "In pattern/PATTERN-pair-remote.md ## Pair teardown, add a bullet saying a run's Worktree-Teardown tears down the bench pairs its darn writer made through internal/pairteardown, with their tips archived first, their driver checks skipped and their live processes ended."
---

### [MEDIUM] A bench pair's held run lock makes the teardown refuse instead of ending it

**Location:** `internal/pairteardown/leftover.go#RemoveBenchPairs`
**Issue:** `td.Run` with `Verdict: EntryEnded` skips the driver checks.
`EndSession` then calls `awaitOrPass` (`internal/pairteardown/teardown.go:299-312`).
It waits for the run lock before it ends the owned processes, and it refuses a holder that no pause covers.
A bench pair whose own driver still runs therefore refuses.
The task's row goes Stuck, and every resume refuses again until the bench run ends by itself.
The test row `ALiveProcessOfABenchPairIsEndedNotRefused` registers a process but holds no run lock, so it does not reach this path.

### [MEDIUM] The bench removal runs under the hub's prime lock

**Location:** `internal/battencli/wire.go:1245`
**Issue:** `NewWorktreeTeardown` (`internal/battenshed/teardown.go`) calls `deps.Remove` inside its second prime-lock hold.
It calls `deps.Push` with no lock held, so that no push's waits hold the lock.
`RemoveBenchPairs` pushes archive tags to origin through `ArchiveOwnerTips` and runs a full teardown per bench pair, run-lock wait included, all inside that hold.
That blocks every other run's prime-lock step on the hub for as long as those pushes and waits take.

### [MEDIUM] The guard admits a bench slug that another task's run owns

**Location:** `internal/fabriccli/roleguard.go#benchPairVerb`
**Issue:** The board entry bounds the new access "so a bench never touches another task's pair".
The guard admits `add` and `remove` for any `<slug>-b<N>`, and only the teardown checks for a board entry or a batten run directory under that name.
A writer can therefore remove another live task's pair.
It can also add a pair for a board task not yet started; that task's Worktree-Create then refuses, and the teardown leaves the pair standing because the board names it.
PATTERN-fabric-boundary keeps fabric from reading the board, and that constraint wins over the answer key's bound for the board half.
The batten run directory is no board record, and `fabriccli` imports `shedrun` already, so the live-run half can be closed.

### [MEDIUM] Merge danger calls a door two-way that pushes outward and deletes

**Location:** `_lym/landing/summary.md`
**Issue:** The rubric makes a `two-way` call a finding when the diff deletes data or pushes anything outward.
`RemoveBenchPairs` pushes owner archive tags to origin and force-removes pairs, ending their live processes.
The blast radius names only the teardown, though the diff also widens the sandboxed-role guard for `add`, `remove` and push.

### [MEDIUM] No test pins the Worktree-Teardown wiring

**Location:** `internal/battencli/wire.go:1242`
**Issue:** No test pins either half of the behaviour this diff adds to the removal half:
the bench pairs go before the task pair, and a failure halts the row with the task pair standing.
`TestWorktreeTeardown_RemoveFailureNamesRemovalHalf` drives only a fake `Remove`, and no battencli or loomcli test reaches `RemoveBenchPairs`.

### [LOW] A half-removed bench pair is skipped without a trace

**Location:** `internal/pairteardown/leftover.go#RemoveBenchPairs`
**Issue:** The `worktree.Prunable` skip and the `ResolveWorktree`-error `continue` drop such a pair silently.
The row succeeds while the pair's records sibling and branches stay behind.

### [LOW] An unreadable run record yields a misleading refusal

**Location:** `internal/fabriccli/roleguard.go#benchPairVerb`
**Issue:** `return err == nil && found && ...` discards the read error.
`sandboxedRoleRefusal` then appends the bench pair way forward, which names the very command the writer ran.

### [LOW] Two guard refusals have no test row

**Location:** `internal/fabriccli/roleguard_test.go`
**Issue:** The bench subtest's rows always write a decodable record and pass one slug.
No row reaches the decode-error refusal or the `len(args) != 1` refusal.

### [LOW] The partial-failure contract of RemoveBenchPairs is unpinned

**Location:** `internal/pairteardown/leftover_integration_test.go#TestRemoveBenchPairs`
**Issue:** The error path returns after `requirePairStanding` and never checks the slugs returned.
The doc comment states the first failure returns with the slugs removed before it, and no row pins that.

### [LOW] The push admission skips the recipe and profile check

**Location:** `internal/fabriccli/roleguard.go#isBenchPairPath`
**Issue:** `benchPairVerb` reads the run record for the darn recipe and the docs profile, and `isBenchPairPath` does not.
A darn writer on any profile may push a pair that has the bench slug form.

### [LOW] PATTERN-pair-remote does not name the bench pair teardown route

**Location:** `pattern/PATTERN-pair-remote.md`
**Issue:** The Pair teardown section lists when the driver checks are skipped and which leftover routes tear pairs down.
The new route skips the checks through an ended verdict for a pair with no entry, and the section does not name it.

## Focus coverage

Each focus entry was reached:
- Slug form vs. another task's pair: F3.
- The flags of `add` and `remove`:
  - `--parent` and `--adopt-local` are refused.
  - `remove --force` and `--remote` stay admitted. They act on the writer's own bench pair only, and `--remote` pushes the archive tags before any branch delete on origin, so they are no finding.
- Push admission: F10. The detached push child `runAdd` spawns carries the writer's strand name and the pair's paths, so it passes `injectedPathsAreOwn`.
- Teardown paths:
  - Only the Worktree-Teardown removal half reaches `RemoveBenchPairs`.
  - Cancel, hand-over and the dobby's leftover teardowns do not reach it. The prime's dobby orphan sweep, which no board entry or batten record stops, archives and removes such bench pairs after the orphan grace. The answer key asks only Worktree-Teardown to remove them, so this is no finding.
  - Against `RemoveOrphanPair`: F1, F6.
- Docs and stencils: no statement the diff leaves stale. The refusal names the new route.
- The change description: F1 for the claim about live processes, and F4.
- Test coverage: F5 and F9. The batten-run-directory skip shares `runNamingReason`, which `TestRemoveOrphanPair` pins.
