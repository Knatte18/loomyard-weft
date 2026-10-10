---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md:123
    summary: 'D8 git forms check option spelling only on words starting with a dash, so a quoted or escaped option such as a quoted upload-pack or output flag reads as a positional word and is accepted, which makes the stated bound that no accepted command writes a ref, config or remote false'
    origin: lens:correctness
  - id: F2
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md:182
    summary: 'D12 rewords only the Master stencil dead rungs, but begin-batch still refuses a terminal dead batch with the text that the recovery is exhausted and the spec row says the same, which contradicts the new retry rung for a retryable dead batch and is not listed among the reworded refusals'
    origin: lens:correctness
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:122
    summary: 'D8 claims the classifier and the D7 guard admit one set, but diff takes exactly one revision argument so the accepted bare form always errors and the working form is rejected, and prune and cleanup are flag-free in the classifier while the guard admits any flag short of apply'
    origin: handler
  - id: F4
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:96
    summary: 'The D7 guard refusal names the sandbox rule and the orch as the one who runs fabric verbs, which is a role and not a verb, a transition or a git command, so the refused fork or recovery strand is left with no step to take'
    origin: lens:error-handling
  - id: F5
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:40
    summary: 'D1 saves the restamped state before the batch loop but does not say whether a failed SaveState is wrapped in ErrAutoRebaseline and blocks the row as Stuck, as every other Run SaveState failure is a returned error'
    origin: lens:error-handling
  - id: F6
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:181
    summary: 'The recovery_retry computation reads git ancestry and a commit range inside the read-only status verb, and D12 does not say what a failed or unresolvable read yields'
    origin: lens:error-handling
  - id: F7
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:198
    summary: 'The PATTERN-refusal-way-forward constraint lists D1, D3, D6 and D12 but not D8, which rewords the ErrAuditNotAcceptable way forward, adds the audit_not_acceptable envelope key and a Master rung, each needing its spec row and a reaching test in the same commit'
    origin: lens:generic
  - id: F8
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:207
    summary: 'D9 rewrites the shuttleengine turn-end, deadline and liveness finalization that PATTERN-completion-signal governs, but Constraints omit it, and the completion-signal tripwire test pins expiredTurnEnd by name'
    origin: lens:generic
  - id: F9
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:112
    summary: 'The D7 rationale says the driver, orch and conflict resolver are the only roles whose stencils name fabric verbs, but only the driver stencil does, so the allowlist for the other two rests on a false premise'
    origin: lens:generic
  - id: F10
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:107
    summary: 'D7 scrubs the strand-name variable only in lyx gate test, but verifytree.Verify and the card verify path run tests with the inherited environment from a Master or recovery session, so a test that execs a real fabric verb would meet the guard there'
    origin: lens:correctness
  - id: F11
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md:14
    summary: 'The Scope bullet says recordedOverviewFrame logs its empty-frame fallback, but D3 says the helper logs nothing and Rebaseline logs the cause at its own call'
    origin: lens:generic
  - id: F12
    severity: NIT
    class: consistency
    location: _lyx/discussion/decision-record.md:174
    summary: 'Recoveries is defined as the count of recovery strands spawned, but a card_amended re-run spawns a strand and keeps the prior count, so the field counts counted spawns only'
    origin: lens:generic
---

### [BLOCKING] D8 git forms accept quoted or escaped options

**Location:** `_lyx/discussion/decision-record.md:123-132`, the Bound at line 140 and the rewritten `AcceptBatchFabricReference` bound at line 141.
**Issue:** D8 matches a flag exactly only for a word that starts with `-`.
`IsReadOnlyCommand` splits a segment with `strings.Fields` on the original text (`internal/fabricengine/refscanner.go:222`) and rejects a quoted word only in first position.
`git ls-remote --heads "--upload-pack=git remote add x u"`, `git log '--output=f'` and `git log \--output=f` therefore parse as a listed flag plus a positional word that does not start with `-`, and every form admits positional words.
The shell hands git the real option, so the command adds a remote or writes a file.
A `$VAR` or a brace expansion in a positional slot does the same.
The `lyx` help form says "unquoted lowercase command words only"; the `git` forms say nothing equivalent.
So the Decision's answer to the round's first focus question is no: an accepted command can add a remote, and the bound "admits nothing outside the forms above" and the rewritten bound "no command the classifier accepts writes tracked content, a ref, config or a remote" are false.
The change downgrades a correctness halt to a policy warning and opens recover-batch's in-line accept, so the bypass is a guard downgrade with a false bound.
**Fix:** State that every word of a `git` form is plain: no quote, backslash, `$`, brace or glob character, else the classifier rejects the command.
Add Acceptance rows rejecting a quoted `--upload-pack=` and a quoted `--output=`, a backslash-escaped option, a brace-expanded option and a `$VAR` positional.

### [MEDIUM] D12 leaves the begin-batch dead refusal and its spec row saying exhausted

**Location:** `_lyx/discussion/decision-record.md:182-183`; `internal/websterengine/beginbatch.go:296`; `contracts/specs/refusal-spec.md:63`.
**Issue:** D12 rewords the Master stencil's two dead rungs and the "recovery is exhausted" sentence there.
`begin-batch` still refuses a terminal `dead` batch with "the recovery of batch %d is exhausted, so end the run stuck naming the batch", and the `report already present` spec row quotes it.
After D12 that remedy is false for a dead batch with `recovery_retry: true`, and the Master stencil tells Master to follow a refusal's remedy as written.
The Constraints name `recovery_exhausted` as D12's only new or reworded refusal, so this one is not tracked either.
**Fix:** Add to D12 that the begin-batch dead remedy and its spec row route a retryable dead batch to `recover-batch`, and name that refusal beside `recovery_exhausted` in the `PATTERN-refusal-way-forward` constraint.

### [LOW] D8's claim that classifier and guard admit one set is false

**Location:** `_lyx/discussion/decision-record.md:122` against line 95.
**Issue:** `lyx fabric diff` is `cobra.ExactArgs(1)` (`internal/fabriccli/weft_verbs.go:360`), so D8's "`diff` alone" is a form that always errors while the working `lyx fabric diff <sha>` is rejected.
D7's guard admits `prune` and `cleanup` with any flag short of `--apply`; D8's classifier admits them with no flag.
The classifier is a strict subset, not the same set, and the criteria pin neither side.
**Fix:** Let the classifier's `diff` form take exactly one non-flag word, and say the classifier admits a subset of the guard's readers built from the same verb names.

### [LOW] The D7 guard refusal gives the refused strand no step

**Location:** `_lyx/discussion/decision-record.md:96`.
**Issue:** The way forward is "the sandbox rule and the orch as the one who runs fabric verbs".
The refusal-spec defines a way forward as a verb, a run transition or a git command; a role is none of them, and a fork or recovery strand has no stated channel to the orch.
**Fix:** Name the step the refused strand takes, for example stop and report the needed fabric action in its report or `stuck_reason` so the orch runs it; keep the orch route for the spec rows.

### [LOW] D1 does not place a failed save after an accepted auto-rebaseline

**Location:** `_lyx/discussion/decision-record.md:40-45`.
**Issue:** `Rebaseline` does not save (`internal/webstercli/rebaseline.go:109` saves); the Decision has Run save before the batch loop, outside "every error the auto-rebaseline returns".
Every other `SaveState` failure in `Run` is a returned error that becomes `producer-hard-failure`, the outcome D1 exists to avoid on this path.
**Fix:** Say a save failure after an accepted rebaseline is wrapped in `ErrAutoRebaseline`, blocks the row as Stuck with the transient re-step text, and Run does not continue on the in-memory restamp.

### [LOW] recovery_retry has no stated result for a failed git read

**Location:** `_lyx/discussion/decision-record.md:181`.
**Issue:** The function needs ancestry and first-parent range reads, in `status` as well as `recover-batch`.
D12 lists the false cases but not a read error or a `RecoveryStartSHA` absent from the store.
**Fix:** State that a failed or unresolvable read yields `recovery_retry: false`, logged, so `status` never fails on it and Master ends stuck as today.

### [LOW] Constraints omit D8 from the refusal-way-forward line

**Location:** `_lyx/discussion/decision-record.md:198`.
**Issue:** D8 rewords the `ErrAuditNotAcceptable` way forward, adds the `audit_not_acceptable` envelope key and a Master rung; the line lists D1, D3, D6 and D12 only.
**Fix:** Add D8 to the constraint.

### [LOW] Constraints omit PATTERN-completion-signal for D9

**Location:** `_lyx/discussion/decision-record.md:207`.
**Issue:** D9 adds a gated at-once branch and an `EndedShells` record at the deadline and liveness ends in `internal/shuttleengine/wait.go`.
`PATTERN-completion-signal` requires `allOutputFilesExist` first on every negative path, and its tripwire test pins `expiredTurnEnd` by name.
**Fix:** Add a constraint: D9 keeps every negative-verdict site behind `allOutputFilesExist`, and the tripwire test's audited sets change in the same commit.

### [LOW] D7 rationale rests on a false premise

**Location:** `_lyx/discussion/decision-record.md:112`.
**Issue:** Over `contracts/stencils`, only `shed/shed-template-driver.md` names a `lyx fabric` verb; the orch and conflict stencils name none.
The allowlist and the `mergeresolve` import for the conflict role are justified by a sentence that is not true.
**Fix:** Restate the rationale truthfully, for example that the orch and conflict resolver are the roles that may merge or drive pairs, or drop the conflict role and its import if it has no fabric business.

### [LOW] D7's environment scrub covers one test entry point

**Location:** `_lyx/discussion/decision-record.md:107-108`; `support-log.md:42`.
**Issue:** `verifytree.Verify` passes `os.Environ()` to its children (`internal/verifytree/verify.go:249`), and a plan's `## verify:` and a card's `**Verify:**` run `go test` from a Master or recovery session.
The support log's claim that the gate is the one place every test process passes through is wrong for those paths.
I found no untagged test that execs a guarded verb, so the harm is unconfirmed.
**Fix:** Scope the claim to `lyx gate test`, or have the verify paths drop the variable too.

### [LOW] Scope bullet and D3 disagree on who logs the empty-frame fallback

**Location:** `_lyx/discussion/decision-record.md:14` against line 68.
**Issue:** Scope says `recordedOverviewFrame` logs its fallback; D3 says the helper logs nothing and `Rebaseline` logs the cause at its own call.
**Fix:** Reword the Scope bullet to say the rebaseline logs the cause and the refusal names it.

### [NIT] Recoveries counts counted spawns only

**Location:** `_lyx/discussion/decision-record.md:174`.
**Issue:** "the count of recovery strands spawned" is false for a `card_amended` re-run, which spawns a strand and keeps the prior count.
**Fix:** Define it as the count of counted recovery spawns.

## Focus verdicts

- D8 positional words: each form states its positional words (revisions, paths after `--`, one remote or object name, patterns), but the rule does not cover a quoted or escaped positional word, so the answer is no (F1).
- D7 guard: settled, not a defect.
  It is the first check of the persistent pre-run on `fabric`, ahead of the weft early return (`internal/fabriccli/weft_verbs.go:61-64`).
  A bypass flag on every subcommand but `push` is refused.
  The nested sync is told apart by the missing parent, since `RunCLIIn` runs `Command()` as the root.
  The `mergeresolve` import has no cycle.
  A refused role's bypass `push` can name any worktree paths, but the Decision states that and the strand-name bound, and Open risks carry it.
- D12 counting: settled, not a defect.
  The count lives on `BatchState`, HEAD is read before `Starter.Start`, `recover-batch` saves the returned record before awaiting, and a failed start counts nothing.
- Refusal ways forward: D12's `recovery_exhausted` and D8's `audit_not_acceptable` name operator steps (a reset, a tree cleanup) that Master may not run, but each gets a Master rung that stops and quotes the refusal, the same shape as the existing `needs_fresh` rung, so they are not defects.
  D7's guard refusal is F4, and D12's begin-batch dead refusal is F2.

## Rejected

- Test-gap demands for a row per variant of D7's allowlist, D8's forms, D9's gated cases, D10, D12's retry predicate, D13's cap and D1/D3/D4/D5 sub-behaviors: the Acceptance criteria name behaviors, not tests, and a new row in an existing table meets them; adding one criterion per variant is the additive growth the rubric warns against.
- Acceptance criterion pinning the guard refusal's text, and the friction note on `recovery_exhausted`: the Constraints already require both, and the reaching test carries them.
- D9 "waiting turn end" read as contradictory: it is the existing term of art (`TestPollEventsTick_GatedWaitingWithOutputFilesIsNotAnArrival`), and the Decision and the criterion agree on it.
- D7 blast radius beyond webster roles: the Rationale states the deny-by-default allowlist, and the Acceptance criteria need no per-role row.
- Auto-mode assumptions missing the D2 and D11 picks: both choices are stated in D2 and D11 and no requirement is lost.
- D12 and D8 operator-only ways forward (Master cannot run the named reset): see the focus verdicts.
