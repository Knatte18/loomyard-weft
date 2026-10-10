---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D8 (git read forms and the AcceptBatchFabricReference bound)
    summary: D8 git forms constrain only dash-words, so git remote -v add and git branch with a name pass as read-only and write, and the stated bound that no accepted command can write is false
    origin: handler
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D7 (bypass bound sentence)
    summary: D7 says bypass mode refuses every other subcommand, but the existing bypass refusal covers only the weft verbs, so fabric with a bypass flag before add or clone would pass a guard that exempts bypass mode
    origin: handler
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D7 (refusal-spec rewording sentence)
    summary: D7 rewords two refusal-spec rows, but many other rows and error texts name a fabric verb the refused roles can no longer run, so PATTERN-refusal-way-forward is breached
    origin: lens:error-handling
  - id: F4
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md D7 (test rule bullet) and Constraints PATTERN-test-isolation
    summary: lyx gate test run from a webster strand passes the strand name to every test, so existing tests that drive add, clone, push or sync through RunCLI fail, and per-test t.Setenv does not cover them
    origin: lens:test-gaps
  - id: F5
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md D7 (guard runs only when fabric is the invoked top-level command)
    summary: configcli's nested sync runs the same Command() as root through RunCLIIn, so the stated discriminator cannot exempt it, and the guard must also sit ahead of the weft pre-run's early return
    origin: lens:correctness
  - id: F6
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D12 (first paragraph, save before the strand starts)
    summary: The record cannot be saved before Start because StrandGUID and run paths exist only after it, no save seam exists in RecoverDeps, and counting a failed start burns the cap with no recovery run
    origin: handler
  - id: F7
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md D12 (stencil sentences) and contracts/stencils/webster/webster-template-master.md lines 59 and 133
    summary: D12 changes only the dead rung of the recover-batch ladder, while the entry-time dead rung and the exhausted-batch sentence still end the run stuck, and status carries no recovery_retry
    origin: handler
  - id: F8
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md D8 (failed evidence refuses with ErrAuditNotAcceptable)
    summary: recover-batch maps only the needs-fresh and delete-referenced sentinels to envelope keys, so the in-line evidence refusal matches no Master ladder rung
    origin: lens:correctness
  - id: F9
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md D1 and the second acceptance criterion
    summary: D1 makes only the step name step-keyed, but followUpCardLanding and the changed-but-not-named and transient texts in rebaseline.go still tell the reader to run lyx webster rebaseline by hand
    origin: lens:generic
  - id: F10
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md D9 (last sentence of the Decision)
    summary: TestTurnEnded pins TurnEndedAfter over an engine without the SessionSignalParser capability, so outstanding-shell cases are a new fake and test, not new rows
    origin: lens:test-gaps
  - id: F11
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D3
    summary: recordedOverviewFrame is also called from fingerprintMismatchWayForward on every mismatch message, so logging inside it repeats the Info line, and that message has its own uncovered no-frame wording
    origin: lens:generic
  - id: F12
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md D1 (Stuck mapping)
    summary: D1 does not say how the producer tells the auto-rebaseline errors from other Run errors, nor whether the entry rebaseline syncs fabric as the verb does
    origin: lens:generic
  - id: F13
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md Notes for the plan writer, D11 candidates
    summary: The reconcile-commit push candidate contradicts the test, whose loom status call reconciles and commits nothing
    origin: lens:generic
  - id: F14
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md D9 (ExpiredShells record)
    summary: The record of every shell a run ends over misses the ungated at-once path and deadline or liveness ends, and the field name ExpiredShells no longer matches what it holds
    origin: lens:correctness
  - id: F15
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D7 and D8 reader lists
    summary: D7 and D8 list different lyx fabric readers with no stated reason
    origin: lens:generic
  - id: F16
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md Acceptance criteria for D7, D8, D12
    summary: Criteria omit refused and passing cases the Decisions state, namely prune and cleanup with apply, shortname with an argument, an unparseable name, recovery_retry false for a moved HEAD or merge-only range, the recoveries value, and git global-option rejections
    origin: lens:test-gaps
  - id: F17
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md D8 (last sentence) and support-log.md Question ledger
    summary: D8 says the git status, lyx and || rows flip in TestIsReadOnlyCommand but only the git status row exists, and the stdin question is listed Open though D4 already answers it
    origin: lens:test-gaps
  - id: F18
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md D8 (ClassifyViolation and RecoverDeps shapes)
    summary: D8 pins a parameter and field type shape the discussion writer is told not to gather
    origin: lens:generic
---

### [BLOCKING] D8 git read forms admit writes

**Location:** `_lyx/discussion/decision-record.md` D8, the `remote -v`, `branch` and `remote get-url` forms and the closing bound.
**Issue:** The rule "every word starting with `-` before a `--` must be one the form lists" says nothing about positional words or subcommands.
I probed both in a scratch repository: `git remote -v add foo <url>` added the remote, and `git branch newbranch -v` created a branch, though the only dash-word in each is listed.
`git status` also refreshes the index.
So the Decision's bound, "no command the classifier accepts can write", is false.
It is the guard that `AcceptBatchFabricReference` trusts and that D8 downgrades from a failed batch to policy.
The acceptance list rejects `git branch -D x` and `git config x y` but pins no bare-name case.
**Fix:** State for each form what positional words it takes.
`remote -v` takes none, `branch` takes a positional word only with `--list`, and `-n` takes digits.
Reword the bound to "writes no tracked content, refs or config", or drop `status`.
Add `git remote -v add x u` and `git branch -v x` to the rejected cases.

### [MEDIUM] D7's bypass bound is false for topology verbs

**Location:** `_lyx/discussion/decision-record.md` D7, "bypass mode refuses every other subcommand".
**Issue:** `--weft-path` and `--warp-path` are persistent flags on the `fabric` command, but the existing pre-run returns early unless the verb is in `weftVerbNames` (`internal/fabriccli/weft_verbs.go:61`).
The bypass refusal for non-`push` therefore covers only the weft verbs.
`lyx fabric --warp-path /x add ...` or `clone` is accepted and ignores the flags today.
A guard written as D7 words it, "a push carrying the bypass flag passes", is bypassed from any slugged role by typing the flag before `add`.
No acceptance row pins a bypass flag with a non-push verb.
**Fix:** Say the guard itself refuses a bypass flag on any subcommand but `push`, and add a row for `add` and `clone` with the flag under a webster strand name.

### [MEDIUM] Refusal rows name verbs the refused roles may not run

**Location:** `_lyx/discussion/decision-record.md` D7; `contracts/specs/refusal-spec.md` lines 69, 121, 146, 153, 154, 155, 242, 246, 247, 269, 277, 300; `internal/gatecli/run.go:57`.
**Issue:** D7 rewords only `fabric sync failed` and `decision: commit failed after the append`.
Other rows hand Master, a recovery strand or an implementer a verb the guard refuses: `fabric merge --continue` and `--abort`, `fabric push`, `fabric commit`, `fabric reconcile`, `fabric shortname <x>` and `fabric add`.
The `gate.yaml absent` text names `lyx fabric reconcile` to the implementer running `lyx gate test`.
This breaches PATTERN-refusal-way-forward for the refusal D7 introduces.
**Fix:** State one rule: any spec row or error text whose way forward is a refused fabric verb says the orch runs it when the refusing session cannot.
Have the plan writer derive the row set by search, not from a two-row list.

### [MEDIUM] The gate's inherited strand name fails existing fabric tests

**Location:** `_lyx/discussion/decision-record.md` D7 test bullet and Constraints; `internal/gatecli/run.go`, `internal/fabriccli/*_test.go`, `internal/hubgeom/geometry_integration_test.go`.
**Issue:** An implementer fork or Master runs `lyx gate test` with its strand name exported, and the test binaries inherit it.
Existing tests that call `fabriccli.RunCLI` or `RunCLIIn` with `add`, `clone`, `push` or `sync` would hit the new guard.
The record's rule of `t.Setenv` per guard test, with no `t.Parallel`, covers new guard rows only, forces many parallel tests serial, and leaves tests that run the `lyx` binary as a subprocess.
**Fix:** Pick one mechanism in the Decision.
Clearing the variable once in the package `TestMain` path is the cheaper option, with `t.Setenv` only in the guard rows.
Add a gate-run-under-a-webster-strand acceptance row.

### [MEDIUM] The nested-sync exemption has no discriminator

**Location:** `_lyx/discussion/decision-record.md` D7, "The guard runs only when `fabric` is the invoked top-level command"; `internal/configcli/configcli.go:527`; `internal/fabriccli/fabric.go:513`.
**Issue:** `configcli` syncs with `fabriccli.RunCLIIn(cwd, w, []string{"sync"})`, which executes `Command()` as the cobra root.
The guard tests drive the same entry.
No property of the cobra tree lets the first pass and the second refuse.
The guard must also run ahead of the early return in the weft pre-run, which the record does not say.
**Fix:** Name the discriminator, for example the guard applies only to a `Command()` mounted under a parent as `main.go` mounts it, or only on the CLI entry.
Say which tree the subcommand-walk test mounts.

### [MEDIUM] D12's save-before-start does not fit the code, and counts a failed start

**Location:** `_lyx/discussion/decision-record.md` D12, first paragraph; `internal/websterengine/recoverbatch.go:219-345`; `internal/webstercli/recoverbatch.go:178-199`.
**Issue:** `recoverSpawn` builds the new record after `Starter.Start`, because `StrandGUID`, `ShuttleRunDir` and `EventsPath` exist only then, and it reads HEAD after `Start` too.
`RecoverDeps` has no save seam; the caller saves after `RecoverSpawnOrAttach` returns.
"Saves both before the strand starts" names no record to save onto and no place to do it.
Counting "a start that fails" also burns one of two recoveries on a transient start failure, the same waste the support log cites in rejecting amendment counting.
**Fix:** State where the pre-start save happens and onto which record, and read `RecoveryStartSHA` before `Start`.
Either count only a started strand, or keep the count and name the cost in the Bound or Open risks.

### [MEDIUM] Master's other dead rungs still end the run stuck

**Location:** `_lyx/discussion/decision-record.md` D12 stencil sentences; `contracts/stencils/webster/webster-template-master.md` lines 59 and 133.
**Issue:** D12 changes the dead rung under `recover-batch`.
The entry-time rung, "`dead` → exhausted, write `outcome: stuck`", and the sentence "A batch whose recovery is exhausted ends the run" are not mentioned.
`recovery_retry` rides on the `recover-batch` envelope, so a Master restarted after a dead recovery that committed can never retry it.
The `recovery_exhausted` refusal also names no envelope key for the rung to match, where the other rungs match `batch_failed` and `needs_fresh`.
**Fix:** Say the entry rung stays stuck, or carry `recovery_retry` on `status`.
Name the refusal's envelope key.

### [MEDIUM] The in-line evidence refusal matches no Master rung

**Location:** `_lyx/discussion/decision-record.md` D8; `internal/webstercli/recoverbatch.go:183-192`.
**Issue:** `recover-batch` maps only `ErrRecoveryNeedsFresh` and `ErrRecoveryDeleteReferenced` to envelope keys.
`ErrAuditNotAcceptable` falls to a plain error, which no ladder rung matches, and Master cannot clean a dirty tree.
**Fix:** Carry a key on that refusal and add a rung that ends stuck quoting it.

### [MEDIUM] Auto-path refusals still name the by-hand rebaseline

**Location:** `_lyx/discussion/decision-record.md` D1 and the second acceptance criterion; `internal/websterengine/rebaseline.go:25,203,214,229,265`.
**Issue:** D1 makes the step name come from deps, which covers `stepRun` only.
`followUpCardLanding` ends "then run `lyx webster rebaseline --card ...`", and the changed-but-not-named and transient texts say "re-run `lyx webster rebaseline`".
The acceptance bullet checks for the absence of `lyx webster run`, which these texts would still pass.
**Fix:** Name those clauses as step-keyed in D1, and assert that no auto-path refusal contains `lyx webster rebaseline`.

### [MEDIUM] The new TestTurnEnded cases are not rows

**Location:** `_lyx/discussion/decision-record.md` D9, last sentence.
**Issue:** `TestTurnEnded` pins `TurnEndedAfter` over an engine without the `SessionSignalParser` capability, and outstanding shells exist only on the other branch.
No websterengine test or fake exercises that branch today.
**Fix:** Say the branch needs a signal-parsing fake and a new test.

### [LOW] D3's log sits in a helper with a second caller

**Location:** `_lyx/discussion/decision-record.md` D3; `internal/websterengine/beginbatch.go:60`.
**Issue:** `recordedOverviewFrame` is also called from `fingerprintMismatchWayForward`, which has its own no-frame wording.
Logging inside the helper repeats the Info line, against "once each".
**Fix:** Log at the `Rebaseline` call site, and say whether the other message names the cause.

### [LOW] D1 does not say how the Stuck mapping identifies its errors

**Location:** `_lyx/discussion/decision-record.md` D1.
**Issue:** `Run` returns plain errors and the producer recognises only `ErrPendingAuditFindings`.
The record does not say whether the entry rebaseline syncs fabric as the verb does after `SaveState`.
**Fix:** State the identification and the sync.

### [LOW] D11's push candidate contradicts the test

**Location:** `_lyx/discussion/decision-record.md` Notes for the plan writer, D11.
**Issue:** The `loom status` call in the test reconciles and commits nothing, so "the reconcile commit's detached push on the `loom status` call" cannot be a dependency.
**Fix:** Drop or rewrite the candidate.

### [LOW] D9's shell record misses ends and its field name is stale

**Location:** `_lyx/discussion/decision-record.md` D9.
**Issue:** An ungated at-once finish returns before the expiry path, so outstanding shells are not recorded, and deadline or liveness ends are not covered.
`ExpiredShells` no longer holds expired shells.
**Fix:** State which ends record shells, and rename the field or say why it keeps the name.

### [LOW] D7 and D8 list different fabric readers

**Location:** `_lyx/discussion/decision-record.md` D7 and D8.
**Issue:** D7 admits `list`, `pairs`, `status`, `diff`, `shortname`, `prune` and `cleanup`; D8 admits four.
**Fix:** Share one list or say why D8's is narrower.

### [LOW] Acceptance criteria omit stated cases

**Location:** `_lyx/discussion/decision-record.md` Acceptance criteria for D7, D8 and D12.
**Issue:** Not pinned: `prune` and `cleanup` with `--apply` refused, `shortname` with an argument refused, an unparseable name passing, `recovery_retry` false for a moved HEAD and for a merge-only range, the `recoveries` value, and rejection of other git global options.
**Fix:** Add them as rows of the existing cases.

### [NIT] Test-name and ledger slips

**Location:** `_lyx/discussion/decision-record.md` D8; `_lyx/discussion/support-log.md` Question ledger.
**Issue:** Only the `git status` row of `TestIsReadOnlyCommand` flips; the `lyx` and `||` rows are new.
The ledger lists the stdin question Open though D4 answers it.
**Fix:** Reword D8 to flips and adds, and mark the ledger entry resolved.

### [NIT] D8 pins a signature shape

**Location:** `_lyx/discussion/decision-record.md` D8.
**Issue:** The writer's stencil says not to gather signatures or interface shapes; `readOnly func(string) bool` is one.
**Fix:** Keep the injection constraint and drop the type shape.

## Rejected

- D11 acceptance criterion cannot fail when no run reproduces (lens:test-gaps) - the Decision and Open risks accept this bound by design.
- D13 boundary rows at exactly the cap, multibyte and unfenced lines (lens:test-gaps) - the planned table can cover them; the Decision states the rule, and over-listing rows is the additive bias the rubric warns against.
- D1 crash re-entry row and D5 positive `cd` row (lens:test-gaps) - folded into the table rows the plan writer derives from each Decision; not a gap in the record.
- D2 and D13 parity with `lyx loom validate-plan` lacks an acceptance row (lens:test-gaps) - stated as a Constraint, which is where a cross-cutting invariant belongs.
