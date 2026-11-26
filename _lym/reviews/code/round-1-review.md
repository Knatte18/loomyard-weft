---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/remoteteardown.go#rerunWayForward
    summary: "A cancel's removal record now names `lym batten cancel <slug> --remove` even where that cancel refuses: when only the records branch on origin is left, `refuseNothingLeft` answers nothing-left, while the replaced `lym fabric remove <slug> --remote` finds that branch and finishes"
    fix: "Make the named re-run work in every state the record can be listed in. Either (a) have `refuseNothingLeft` in internal/battencli/cancel.go treat a records branch of the pair on origin as something left (read it the way `originPairBranch` in internal/fabricengine/pairbranch.go does, after the task-branch probe), so `lym batten cancel <slug> --remove` reaches `removePairAndBranch` and its `Remove` finishes the origin records branch; or (b) name the caller's command only where it works, keeping `lym fabric remove <slug> --remote` for the failed origin-delete steps of `failedRemoteSteps`. Add a test that drives the cancel's re-run from the state with only the origin records branch left and asserts it removes that branch."
  - id: F2
    severity: BLOCKING
    class: consistency
    origin: lens:code-affected-set
    location: internal/fabricengine/doc.go:242
    summary: "The added doc.go sentence documents battencli's command (\"`lym batten cancel <slug> --remove` finishes even once nothing of the pair remains\"), which PATTERN-code-conventions forbids in a package's doc.go, and the claim is false: the cancel refuses nothing-left once its records and claim are gone"
    fix: "In internal/fabricengine/doc.go, delete the line \"A command such as `lym batten cancel <slug> --remove` finishes even once nothing of the pair remains.\" and keep the paragraph to fabricengine's own behaviour: a removal's job and record carry the command its caller names, and a removal whose caller names none is re-run by `lym fabric remove <slug> --remote`."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/remoteteardown.go#recordsArchiveWayForward
    summary: "For a removal carrying Rerun, the \"when origin still holds a branch of the pair\" condition is dropped on the claim that the caller's command finishes whether or not origin holds the pair; the cancel refuses nothing-left once its records and claim are cleared and origin holds nothing, so the unconditional way forward and its comment are wrong"
    fix: "In recordsArchiveWayForward, keep the \"when origin still holds a branch of the pair, \" prefix for every remoteTeardownRemove job (revert the `&& job.Rerun == \"\"` part of the condition and the added comment line), unless F1's fix makes the cancel succeed in that state; then reword the comment to the condition the cancel really meets. Update the expected text in TestRemove_PendingRecordNamesTheRerunItsCallerGave (internal/fabricengine/remoteteardown_rerun_integration_test.go) and the summary's What-changes bullet to match."
  - id: F4
    severity: MEDIUM
    class: decision
    origin: lens:code-scope
    location: internal/fabricengine/remoteteardown.go#rerunWayForward
    summary: "The board entry asks to name `lym batten cancel <slug> --remove` when the record's pair is gone, or when it came from a batten cancel; the diff builds only the second, and the narrowing for a pair-gone record written by `lym fabric remove` (or the dobby, beam-fork or batten teardown-row callers) is recorded by no Added after Discussion entry"
    fix: "Leave the code in place and defer this finding to the run's parent: recording the narrowing, or refusing it so the pair-gone case is built, is the parent's call through `lym loom decision add`, which lies outside the fixer's write surface."
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/fabricengine/remove.go#removePair
    summary: "In the board entry's own case, the pair gone locally and on origin, the named cancel succeeds (it tolerates ErrPairNotFound) but nothing clears the pending record: Remove refuses before holdTeardown so no child runs, and only the child calls clearPendingTeardown, so `lym fabric status` lists the record with the same way forward forever"
    fix: "When removePair's refusal is ErrPairNotFound from refuseWhenNothingRemains (origin was asked and holds nothing), clear the slug's pending-teardown record under the teardown lock, after pushing any local archive tag the record's WeftTip names (pushOwedArchiveTags does this for a job); or have the cancel clear it once its removal reads the pair as gone. Add a test: a pending remove record whose pair is gone everywhere is not listed by PendingTeardowns after the named re-run."
  - id: F6
    severity: MEDIUM
    class: consistency
    origin: lens:code-affected-set
    location: pattern/PATTERN-pair-remote.md:100
    summary: "The Pair teardown background still says the chokepoint guard flags a four-argument `Remove` call; the diff makes it five (removeArgCount = 5)"
    fix: "In pattern/PATTERN-pair-remote.md line 100, change \"four-argument\" to \"five-argument\"."
  - id: F7
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/fabricengine/teardownlock_integration_test.go#TestRemove_TeardownLockAgainstThePairsPush
    summary: "No test pins that Remove's held-teardown-lock refusal names the caller's rerun (remove.go lockRerun branch); every Remove call under a held lock passes an empty rerun"
    fix: "Add a subtest to TestRemove_TeardownLockAgainstThePairsPush, beside the discard's DiscardWhoseWaitRunsOutNamesTheGivenRerun, that holds the slug's teardown lock shared and calls h.Topology.Remove(h.Location, slug, false, false, \"lym batten cancel \"+slug+\" --remove\"), asserting the error wraps ErrPairPushRunning and its text holds \"re-run `lym batten cancel <slug> --remove`\" and not `lym fabric remove`."
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/pairteardown/teardown.go:247
    summary: No test pins that the pairteardown remove closure forwards Request.Rerun to Topology.Remove; dropping req.Rerun there passes every test
    fix: "Extend an existing pairteardown test that runs the non-local remove path through New (one the reach report lists, such as internal/fabriccli.TestCommand_CleanupRemoveScenario's pairteardown-side counterpart, or a tier-1 test in internal/pairteardown) so it passes a Request with Rerun set and a teardown child that ends, and asserts the slug's pending record names that Rerun."
  - id: F9
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/landing/summary.md
    summary: "The Merge danger blast radius names only fabric, while the diff changes pairteardown's Request and battencli's cancel way forward, and the What-changes bullet repeats the false claim that the caller's command finishes whether or not origin still holds the pair"
    fix: "In _lym/landing/summary.md, widen the blast radius to fabric, pairteardown and batten cancel, and reword the recordsArchiveWayForward bullet to match the code after F1 and F3 are fixed."
---

### [BLOCKING] A cancel's removal record names a cancel that refuses

**Location:** `internal/fabricengine/remoteteardown.go#rerunWayForward`, with `internal/battencli/cancel.go:783`

**Issue:** `failedRemoteSteps` names `rerunWayForward(job)` for a failed origin weft delete (`remoteteardown.go:273`). For a cancel's job, that is now `lym batten cancel <slug> --remove`.
Take a cancel whose task branch is gone locally and on origin:
- `removePairAndBranch` skips the discard.
- The cancel removes the run's records and clears the claim.
- The detached child then fails the origin delete of the records branch.

The re-run of the cancel reaches `refuseNothingLeft` (`cancel.go:735-769`).
- `otherLeft` is false: `parts.RecordsBranch` reads only the local branch.
- `TaskBranchOnOrigin` is false.

So the cancel answers "nothing of <slug> is left to cancel".
The replaced `lym fabric remove <slug> --remote` asks origin through `originPairBranch` (`pairbranch.go:164-176`), finds the records branch, and finishes.
The change swaps a working way forward for a refusing one.

### [BLOCKING] fabricengine doc.go documents batten's command, and wrongly

**Location:** `internal/fabricengine/doc.go:242`

**Issue:** PATTERN-code-conventions says a package's `doc.go` documents only that package.
The added sentence states how `lym batten cancel` behaves, and the statement is false in the state F1 and F3 describe.

### [MEDIUM] The origin condition is dropped on a false premise

**Location:** `internal/fabricengine/remoteteardown.go#recordsArchiveWayForward`

**Issue:** Take a removal whose branches never reached origin and whose cancel already cleared its records and claim.
The cancel's re-run refuses nothing-left (`cancel.go:753-768`).
The dropped condition "when origin still holds a branch of the pair" was what told the reader this state needs nothing more.
The new comment "a command its caller names finishes whether or not origin holds the pair" is untrue for the one caller that names one.

### [MEDIUM] The pair-gone half of the board entry is narrowed without a recorded decision

**Location:** `internal/fabricengine/remoteteardown.go#rerunWayForward`

**Issue:** The entry's `## Idea` says to name the cancel "When the record's pair is gone, or the record came from a batten cancel".
The diff builds only the second condition.
`_lym/landing/summary.md` justifies leaving out the first on the writer's own authority: a slug may have no batten run, and the check would cost a network call.
The record holds no `Added after Discussion` entry for it, so the parent must record or refuse the narrowing.

### [MEDIUM] Nothing clears a pending record once its pair is gone

**Location:** `internal/fabricengine/remove.go#removePair`

**Issue:** This is the state the board entry's `## Seen` describes: `lym fabric remove` answered "nothing of … remains".
In that state `refuseRemoval` returns `ErrPairNotFound` before `holdTeardown`, so no teardown child is spawned.
`clearPendingTeardown` is called only by the child (`remoteteardown.go:508,535`).
The cancel's `removePairAndBranch` tolerates `ErrPairNotFound` (`cancel.go:786`) and goes on, yet the record stays listed with the way forward the user just followed.
The named re-run therefore never retires the record it is printed in.

### [MEDIUM] The pattern background still says four arguments

**Location:** `pattern/PATTERN-pair-remote.md:100`

**Issue:** The guard's constant and doc comment moved to five arguments, but the background that describes the guard still says four.

### [MEDIUM] Remove's held-lock refusal naming the caller's rerun is untested

**Location:** `internal/fabricengine/teardownlock_integration_test.go#TestRemove_TeardownLockAgainstThePairsPush`

**Issue:** The `lockRerun` branch the diff adds in `removePair` has no test that would fail if it named `lym fabric remove` instead.
The discard's equivalent has one (`DiscardWhoseWaitRunsOutNamesTheGivenRerun`).

### [LOW] pairteardown's forwarding of Rerun is untested

**Location:** `internal/pairteardown/teardown.go:247`

**Issue:** `TestCancelRun` checks the request handed to its fake `RemovePair` and nothing past it.
No test passes a `Rerun` through `pairteardown.New` to `Topology.Remove`.

### [LOW] The change description's blast radius and one claim are off

**Location:** `_lym/landing/summary.md`

**Issue:** The blast radius `fabric` leaves out `pairteardown`'s `Request` and batten cancel's way forward, both of which the diff changes.
The bullet "since the caller's command finishes whether or not origin still holds the pair" is the false premise of F3.
The `two-way` door holds: an older binary ignores `rerun` on a `remove` job, and a newer one falls back on an empty one.

## Rejected

- code-tests LOW, a `WeftTip` on the new `TestPendingTeardowns` row: the condition-dropping branch is already pinned by `TestRemove_PendingRecordNamesTheRerunItsCallerGave`, so a tier-0 copy would duplicate it.
- The `cancelCommand(slug, true)` argument in `removePairAndBranch` was checked against the `rerun` the function receives. The only caller is `clearRunPhase`, under `remove`, so the two name the same command; `rerun` carries `%q` quotes that a command in backticks must not. Not a defect.
