---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/02-shell-result-unread-wording.md
    summary: card 2's reworded warning drops the shell's label, signal and time outstanding from its content list, and the webstercli test asserting outstanding for 1m30s in the friction note is neither on the card nor matched by its grep phrases
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/plan/06-contract-evidence-names-the-writer.md
    summary: card 6 leaves the RecoverSpawnOrAttach refusal prefix failed on contract file(s) a fork wrote last unchanged, so a recovery-kept finding still reads a fork wrote, against D3
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/plan/12-answer-symbols-read-the-classifier.md
    summary: answerSymbols calling classifyAmbiguity per call re-reads and re-classifies an ambiguous answer's files on every call, including per reference line inside coveredByTarget, against D6's one read per candidate file
  - id: F4
    severity: MEDIUM
    class: scope
    location: _lyx/plan/04-verify-gate-way-forward.md
    summary: card 4 pins the verifyGateReprompt helper in an unnamed existing test but no test pins that Run puts the renderer on the verify entry it hands StartMaster
  - id: F5
    severity: LOW
    class: scope
    location: _lyx/plan/09-build-constraint-partition-classifier.md
    summary: card 9's classifier table has no row for a constraint line that cannot be parsed, which its Intent and the acceptance criteria both name
  - id: F6
    severity: LOW
    class: scope
    location: _lyx/plan/06-contract-evidence-names-the-writer.md
    summary: the ArchiveRunAfterReset doc in internal/websterengine/archive.go still says an uncleared contract file a fork wrote last, and the symbol is not on card 6
---

### [MEDIUM] Card 2's reworded warning can break a webstercli test the card does not list

**Location:** `_lyx/plan/02-shell-result-unread-wording.md`, Intent and Tests.

**Issue:** The Intent lists what `expiredShellWarning` says: the shell was still running when the run ended, the run never read its result, lyx did not stop the shell, what ends it and the final outcome.
It leaves out the shell's label, its signal and its time outstanding, all of which the current string carries (`internal/websterengine/triage.go:84`, "background shell `%s` (reported by the %s signal) was still running, outstanding for %s, ...").
An implementer who follows the list literally drops "outstanding for <d>".
`TestRunCmd_DiedMasterNotesExpiredShellOutcome` in `internal/webstercli/verbs_test.go:1035-1041` reads the `webster-background-shell` friction note and asserts it contains `"outstanding for 1m30s"` and "`sleep 9999`".
That file is not under card 2's `**Edit:**`, and the card's grep phrases, "may still be running in the session" and "was still running, outstanding for", do not match it, so it would first fail at the plan-level gate, which is what the plan's own `wording-cards-grep-the-old-text` decision exists to prevent.
The ImpactSummary's "read by their three tests" undercounts for the same reason.

**Fix:** State in the Intent that the warning keeps the shell's label, signal and time outstanding ("outstanding for <d>") and only adds the never-read clause.
Or, if the wording may drop them, add `internal/webstercli/verbs_test.go` to `**Edit:**` and grep for "outstanding for" as well.
Drop the test count from the ImpactSummary.

### [MEDIUM] Card 6 leaves "a fork wrote last" in the recover-batch refusal prefix

**Location:** `_lyx/plan/06-contract-evidence-names-the-writer.md`, Intent paragraph on `RecoverSpawnOrAttach`.

**Issue:** `RecoverSpawnOrAttach` builds its refusal as `"webster: batch %02d failed on contract file(s) a fork wrote last: %s"` around `contractDeleteClause` (`internal/websterengine/recoverbatch.go:481`).
Card 6 rewords `contractDeleteClause` and `noteWroteLast` per path, but says of `RecoverSpawnOrAttach` only that it adopts the new shapes with the `rm` step and rerun unchanged.
The prefix keeps "a fork wrote last", so a refusal over a contract file only a recovery session wrote after Master still reads "a fork wrote".
D3 says the refusal text "name[s] the last writer as a fork or a recovery session, so a recovery-kept finding never reads 'a fork wrote'".
The planned `recovery wrote last` case asserts the refusal names a recovery session, which passes with the stale prefix still present.

**Fix:** Have card 6's Intent reword that prefix, for example "failed on contract file(s) a fork or a recovery session wrote last", or drop the writer from the prefix since the clause now names it per path.
Have the `recovery wrote last` case of `TestRecoverSpawnOrAttach_ContractFileEvidence` also assert the refusal does not contain "a fork wrote".

### [MEDIUM] The classifier runs once per `answerSymbols` call, not once per answer

**Location:** `_lyx/plan/12-answer-symbols-read-the-classifier.md` Intent; `_lyx/plan/00-overview.md` Shared Decision `classifier-takes-the-worktree-root`.

**Issue:** The shared decision has every policy call `classifyAmbiguity(worktreeRoot, candidates)` itself, and card 12 puts that call inside `answerSymbols`.
`answerSymbols` is called in loops: per file and member in `planGatePass` (`internal/planglyph/plangate.go:67`), and in `coveredByTarget` (`internal/planglyph/callercoverage.go:489`), which `callerCoverageFindings` calls for every reference line of every subject, iterating every admissible card's targets (`callercoverage.go:111-118`).
Each call re-reads every candidate file and re-runs up to the pinned pairs times two cgo values times sixteen tag assignments of `MatchFile` per candidate.
D6 bounds the cost as "one read per candidate file", and its Rationale prices the guard on that bound.
No card decides a per-pass memo, so the bound the decision record states does not hold for the plan-gate passes.

**Fix:** Classify each ambiguous answer once per pass and read the result from there.
For example, have card 12 build a classification map keyed by resolve target once at the top of `planGatePass`, `LaterDeleteReferences` and `callerCoverageFindings`, and pass that map rather than `worktreeRoot` to `answerSymbols`, `buildEditRegions` and `coveredByTarget`.
Then update the re-sign arrows and the Shared Decision to match.
Alternatively, state in card 9 that `classifyAmbiguity` memoizes per candidate set within one pass, and say who owns the cache's lifetime.

### [MEDIUM] Card 4 does not pin that `Run` wires the re-prompt onto the verify entry

**Location:** `_lyx/plan/04-verify-gate-way-forward.md`, Tests.

**Issue:** The acceptance criterion is "Webster's `verify` entry's re-prompt opens with `Gate findings recorded at` and names the fixer-fork step and the verify-fix prompt path".
Card 4's tests pin `verifyGateReprompt` itself "as a row of an existing test" in `verifygate_test.go` without naming the test.
None of that file's tests is a table over renderers (`TestVerifyGate_*`, `TestVerifyGateStuckReason_*`, `TestReadModulePath`).
Nothing asserts that `Run` sets `Reprompt` on the entry it appends, which is the behavior D2 changes.
Dropping the `Reprompt:` line from `Run` would pass every test the card plans.
The public-surface test already exists: `TestRun_VerifyGate` in `internal/websterengine/runlevel_git_test.go`, whose `verifyGateOf` helper returns the verify entry `Run` handed `StartMaster`.

**Fix:** Add `internal/websterengine/runlevel_git_test.go` to card 4's `**Edit:**`.
Assert in `TestRun_VerifyGate` that `verifyGateOf(t, fx).Reprompt` is non-nil and its line for a findings path is one line, opens with `Gate findings recorded at <path>` and names the verify-fix prompt path.
Drop the unnamed helper-level row.
The plan's `go test -tags integration ./...` already runs that file.

### [LOW] Card 9's table has no unparseable-constraint row

**Location:** `_lyx/plan/09-build-constraint-partition-classifier.md`, Tests.

**Issue:** The Intent says "a constraint line that cannot be parsed yields not partitioned", and the acceptance criteria say "a candidate file that cannot be read or parsed ... is not partitioned".
The planned table covers an unreadable file but has no row with a malformed `//go:build` line.

**Fix:** Add a row: a candidate whose `//go:build` line does not parse (for example `//go:build linux &&`) beside a `//go:build !linux` file is not partitioned, with each candidate still described.

### [LOW] `ArchiveRunAfterReset`'s doc keeps the fork-only wording

**Location:** `_lyx/plan/06-contract-evidence-names-the-writer.md`, `**Edit:**`.

**Issue:** The doc comment of `ArchiveRunAfterReset` (`internal/websterengine/archive.go:167`) says the archive keeps "an uncleared contract file a fork wrote last".
After card 6 a recovery session's last write keeps the finding too, so the comment goes stale.
Card 6 lists `checkPendingFindings`, whose doc carries the same phrase, but not `ArchiveRunAfterReset`.

**Fix:** Add `internal/websterengine#ArchiveRunAfterReset` to card 6's `**Edit:**` and reword its doc to "a fork or a recovery session wrote last".

## Focus verdicts

- Card 1, the removed fields: not a defect.
  `startedFresh` and `gatedArrival` are read only in `run.go`, `wait.go`, `run_resume_test.go` and `waiting_test.go` (the `TestPollEventsTick_OutstandingBackgroundWork` columns, the deleted `TestWait_GatedFreshRunWithOutstandingShellEvaluatesGateAtOnce`, and the `fresh` column of `TestWait_RecordsEndedShells`), and the card names every one.
  No other shuttleengine test expects the fresh-gated at-once exit.
  The shell rows in `waitshadow_test.go`, `attach_asking_test.go` and `hold_test.go` are ungated or missing an output file.
  The corpus test replays readings only.
  `awaitedShell` keeps its reader in `waitmark.go`'s `shellWaitActive`, as the card says.
- Card 4, the error path: not a defect.
  The card now takes `fixPromptPath` as a parameter that `Run` already resolves and writes at `runlevel.go:678-683` before `NewVerifyGate` at line 719, so no error arises.
  Every caller of the three re-signs is listed: `Run`, `verifygate_test.go`, `verifygatereport_test.go`, `verifygate_integration_test.go` and `webstercli/verify_parity_integration_test.go`.
  `loomshed.NewVerifyGate` is a different function.
  The open gap is the test of the wiring, F4.
- Cards 10, 11 and 12, the bound: each Intent carries a "Bound on the weakened guard" sentence naming the provable-never-coexist rule, the cap and read or parse fallback, the toolchain-free tags, the Found-only Rename old side and the Delete direction.
  That settles the attack-surface question.
  The cost of the classifier those cards call is F3.
