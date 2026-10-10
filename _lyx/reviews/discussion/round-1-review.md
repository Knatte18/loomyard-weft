---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D12
    summary: The retry test HEAD past StartSHA cannot tell a recovery that committed from one that did not, because a recovery inherits the original fork bracket's StartSHA
    origin: lens:correctness
  - id: F2
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D7
    summary: The deny-by-default role allowlist refuses the slug-bearing orch of a nested pair, the very role D7 names as the one who runs fabric verbs, and the bound is silent on a missing or malformed strand name and on verbs added later
    origin: lens:generic
  - id: F3
    severity: BLOCKING
    class: design
    location: _lyx/discussion/decision-record.md D8
    summary: The bound says static shape only, but the git readers it admits (log, show, diff with --output; ls-remote --upload-pack; --ext-diff; branch without a defined list flag) and a quote-blind --help match let a writing command classify as read-only
    origin: lens:correctness
  - id: F4
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D12
    summary: The Recoveries count is not carried onto the fresh BatchState recoverSpawn builds, and the record does not say rebaseline keeps it or that reset to batch-start clears it, so the two-recovery bound is unspecified
    origin: lens:error-handling
  - id: F5
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md D12
    summary: D12 adds a refusal to a path that respawns without limit today, for operator calls and for a dead fork batch with no commits, and leaves the no-commit refusal unnamed with no spec row
    origin: lens:error-handling
  - id: F6
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D9
    summary: D9 counts a gated run's turn end whenever the output files exist, dropping the guard against files left over from an earlier arrival, and the bound names only the stale-shell risk
    origin: lens:generic
  - id: F7
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md D1 and Acceptance criteria
    summary: The rebaseline warning has no sink, since only a done outcome carries RunResult.Warnings and the shed adapter forwards none, and D1 does not say the restamped state is saved before the batch loop
    origin: lens:error-handling
  - id: F8
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md D1, D2 and Acceptance criteria
    summary: D2 gates only done-card edits while D1's bound refuses other edits at Webster, the rationale overstates the gate, and no criterion covers the in-flight amendment path or the refusals the D1 bound lists
    origin: lens:test-gaps
  - id: F9
    severity: MEDIUM
    class: consistency
    location: _lyx/discussion/decision-record.md Constraints, friction-capture line
    summary: The line says D7's refusal writes a friction note, but fabriccli is outside the modules PATTERN-friction-capture binds and no Decision describes friction plumbing there
    origin: lens:error-handling
  - id: F10
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md Goal and _lyx/discussion/support-log.md Interview item 1
    summary: The Goal pins a tally of twelve defects while listing thirteen, and Interview item 1 says the Goal has no bullet for 476 when it already names it
    origin: lens:generic
  - id: F11
    severity: LOW
    class: scope
    location: _lyx/discussion/decision-record.md Acceptance criteria
    summary: Several Scope bullets and Decision parts have no criterion that pins them, among them the add --help push text, the implementer stencil sandbox rule, the fixer-prompt marker, the D3 absent-copy cause and the D9 ungated, awaited and deadline cases
    origin: lens:test-gaps
  - id: F12
    severity: LOW
    class: decision
    location: _lyx/discussion/support-log.md Question ledger and _lyx/discussion/decision-record.md D7
    summary: The fabriccli to mergeresolve import question and its fallback live only in the support log, which Plan-Write never reads, and the fallback strains PATTERN-agent-name
    origin: lens:generic
  - id: F13
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D11
    summary: The candidate-cause list is a plan-writer head start that pins perishable timeout values, and the repeated-run criterion cannot go red to green
    origin: lens:generic
  - id: F14
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D8 and D5
    summary: D8 splits segments as D5 does, but D5 does not say its splitter is quote-aware, which the existing classifier pins, and D8 and D9 flip pinned test rows the record never names
    origin: lens:generic
---

### [BLOCKING] D12's commit test measures the whole bracket, not the recovery

**Location:** `_lyx/discussion/decision-record.md` D12, Acceptance criteria (dead batch with commits).
**Issue:** D12 retries a dead batch "whose recovery left commits (HEAD past the batch's `StartSHA`)".
`recoverSpawn` (`internal/websterengine/recoverbatch.go`) deliberately makes a recovery record inherit the original fork bracket's `StartSHA`.
HEAD is therefore past `StartSHA` whenever the stuck fork committed, even if the dead recovery committed nothing.
A no-progress recovery is retried, and the Bound "only when the prior one committed" is not what the test checks.
**Fix:** Name the baseline the test compares against, for example the HEAD recorded when each recovery was spawned, and make the acceptance row distinguish a fork that committed from a recovery that did.

### [BLOCKING] D7's allowlist refuses the orch it names as the fabric runner

**Location:** `_lyx/discussion/decision-record.md` D7, Auto-mode assumptions.
**Issue:** `internal/hubgeom/parent.go` (`decideParent`) forms a parent's name as `<shortname>:<slug>:orch` whenever the parent worktree is a pair, not the prime.
That name has a slug and a role outside the driver and conflict allowlist, so the refusal fires on the orch that D7's own refusal text calls the one who runs fabric verbs.
The verb side is a closed list of sixteen names, so a mutating verb added later is unguarded; that contradicts "deny by default".
The Bound says nothing about what an unset or unparseable strand name lets through; `configcli`'s model guard lets both pass, and the record's wording only implies it.
**Fix:** Allow `agentname.RoleOrch` whatever its slug, or state why a nested-pair orch is refused.
Key the verb side on "every verb not on the read-only list", or require a test that every registered fabric subcommand is classified.
State that an empty or unparseable name passes, and add an acceptance row for the orch name.

### [BLOCKING] D8's bound does not cover the write-capable flags of the readers it admits

**Location:** `_lyx/discussion/decision-record.md` D8, Acceptance criteria (classifier line).
**Issue:** D8 downgrades a fabric reference to a warning when the classifier accepts its command, which weakens an audit guard, and bounds that as "static shape only".
The git readers are admitted with no flag limits: `log`, `show` and `diff` take `--output=<file>`, which writes; `ls-remote --upload-pack=<cmd>` runs a command; `diff --ext-diff` and `cat-file --textconv` can run configured programs; "`branch` with a list flag" is undefined against `-d`, `-D` and `-m`.
`config --get` is read-only, but the record does not exclude `config` without `--get`.
The `--help` / `-h` test is a word match; the current segment code uses `strings.Fields`, which ignores quoting, so `lyx fabric add "x --help"` or a `--help` after `--` or as a flag value would classify as a reader while the verb runs.
A command behind `env` or `xargs` stays rejected because the first word must be a listed reader, which the record should state.
The criteria's reject rows (`lyx fabric add x`, `cat a > b`, `git push`) test none of these edges.
**Fix:** List the refused flags per git reader, define the accepted `branch` flags exactly, require the `--help` match to be token-aware and honour quotes and `--`, and add a reject row for each.

### [MEDIUM] D12's count is not carried, kept or cleared by any stated step

**Location:** `_lyx/discussion/decision-record.md` D12.
**Issue:** `recoverSpawn` builds a fresh `BatchState` and copies only `AmendedCards`, `AuditWarnings`, `SuspectPaths` and `ForkTranscripts` from the prior.
Unless `Recoveries` is carried and incremented there, every recovery record starts at zero and `Recoveries < 2` never trips.
Rebaseline keeps batch records, so the count survives it, and `reset --to batch-start` discards the record, which `recovery_exhausted`'s way forward relies on; the Decision states neither.
**Fix:** State that each recovery spawn carries `prior.Recoveries + 1` and saves it with the spawn record, that rebaseline leaves it and that a batch-start or start reset clears it.

### [MEDIUM] D12 adds a refusal to a path the operator had and does not mark it

**Location:** `_lyx/discussion/decision-record.md` D12, Acceptance criteria (dead batch without commits).
**Issue:** `RecoverSpawnOrAttach` has no dead-prior refusal today and respawns over any terminal prior but a failed one with uncheckable findings.
D12 refuses a third recovery and a dead batch without commits for every caller, including an operator-run `recover-batch`, and a fork batch that classifies dead without ever committing would now be refused where it was recoverable.
`recovery_exhausted` is named only for the third attempt; the no-commit refusal has no name or way forward, though PATTERN-refusal-way-forward needs a spec row for each.
**Fix:** State that the gate applies to a recovery-kind prior only, or say that the operator path loses the respawn and name the way forward.
Name the no-commit refusal or fold it under `recovery_exhausted`, and give it a spec row.

### [MEDIUM] D9's bound leaves a gated run's leftover files unaddressed

**Location:** `_lyx/discussion/decision-record.md` D9, Open risks.
**Issue:** D9 is not a contradiction of 7462c1dc5 for a recovery strand: `Classify` takes a present report first and reads the turn end only when none exists, so "files present counts at once, files missing waits on a shell of either signal" matches the recovery rule.
For a shuttle run, however, `pollEventsTick` deliberately does not count a gated run's waiting turn end when the files exist, since they "may be left over from an earlier arrival while the session works on in the background".
D9 says "gated or not" and extends the existing exception from unawaited payload shells to transcript-reported ones, which are the weaker signal.
The bound and the Open risks cover only a stale shell holding a run open, not a gated turn end counted on stale files and the gate run mid-work.
`ShellWaitExpires` and `TurnEndedAfter` take no output-file input, and after D9 the shell expiry is gone from both; "follows the same rule" should say so.
**Fix:** State what bounds the gated stale-file case or keep the deferral for it, and say the expiry leaves `ShellWaitExpires`.

### [MEDIUM] D1's warning has no sink and the restamp has no stated save

**Location:** `_lyx/discussion/decision-record.md` D1, Acceptance criteria (first row).
**Issue:** D1 carries the accepted rebaseline "as a `RunResult` warning", and the first criterion says the step records a warning.
`runlevel.go` notes that only a done outcome carries `RunResult.Warnings`, and `shedadapters/webster.go` forwards no warnings: `Done` returns the summary path and `Stuck` the reason.
After a rebaseline that ends stuck or errored, only the log line remains, and after done nothing reaches the step record.
D1 also does not say the restamped state is saved before the batch loop; without it a crash re-runs the rebaseline and resets an in-flight amendment's `Rendered` flag.
**Fix:** Name the real sink (the summary or the Stuck reason), rewrite the criterion to match, and state that the restamp is saved before the loop.

### [MEDIUM] D1 and D2 agree on done cards only, and the criteria do not cover the rest

**Location:** `_lyx/discussion/decision-record.md` D1 Bound, D2 Rationale, Acceptance criteria.
**Issue:** Both accept a follow-up card and refuse a done-card edit, and that matches `Rebaseline`.
The D2 gate reports only done-card edits, while D1 also refuses a begun batch's card-set change, a failed batch with uncheckable findings and an overview change outside the Card Index, so a Plan-Review round can pass an edit that then blocks at Webster.
D2's Rationale, "turns an unwinnable edit into a finding at the round", is true only of the done case.
No criterion covers the in-flight amendment (accepted, recorded unrendered, re-run by recovery), which Open risks names, or the refusals the D1 bound lists.
**Fix:** Say which refused edits the gate leaves to Webster, or extend the check; add an acceptance row for the in-flight amendment and one Stuck row per listed refusal.

### [MEDIUM] The friction-capture constraint demands plumbing no Decision describes

**Location:** `_lyx/discussion/decision-record.md` Constraints (PATTERN-friction-capture and PATTERN-refusal-way-forward lines).
**Issue:** The refusal-way-forward line puts `fabriccli` outside the bound modules, but the friction line says D7's refusal writes a friction note "as every refusal does".
PATTERN-friction-capture binds loom halts and webster refusals, `fabriccli` has no friction code, and no criterion tests a note.
**Fix:** Drop D7 from the friction line, or add a D7 part that says how `fabriccli` resolves the friction directory, with a criterion.

### [LOW] The Goal pins a stale tally and the Interview contradicts the Goal

**Location:** `_lyx/discussion/decision-record.md` Goal; `_lyx/discussion/support-log.md` Interview item 1.
**Issue:** The Goal says "Twelve defects" and lists ten issue numbers plus three named items, which is thirteen and matches the thirteen Scope bullets and Decisions D1 to D13.
Interview item 1 says the Goal has no bullet for the 476 item, but the Goal already names it.
The count is a perishable tally.
**Fix:** Drop the count from the Goal and correct Interview item 1.

### [LOW] Some Scope bullets and Decision parts have no pinning criterion

**Location:** `_lyx/discussion/decision-record.md` Acceptance criteria.
**Issue:** No criterion pins the `lyx fabric add --help` push text, the implementer stencil's sandbox rule, the Plan-Burler fixer-prompt marker (only the rubric is covered), the D2 "no recorded hashes" report-nothing bound, the D3 absent-copy cause (only the unparseable cause is covered), the D7 `prune` and `cleanup` pass-without-`--apply` rows, the D9 ungated, awaited-prefix, fork and deadline cases, or D13's inline-span form.
"`ShellWaitExpires` agrees for a recovery strand" is untestable as worded; it needs the report-present and report-missing cases.
**Fix:** Add one row per gap to the existing criteria, and rewrite the recovery clause as two cases.

### [LOW] An open design question sits where the plan writer cannot read it

**Location:** `_lyx/discussion/support-log.md` Question ledger; `_lyx/discussion/decision-record.md` D7.
**Issue:** Whether `fabriccli` may import `mergeresolve` without a cycle is open only in the support log, and its fallback moves `ConflictRole` to `agentname`.
Plan-Write never reads the support log, and PATTERN-agent-name has the spawning module own its role constant, so the fallback needs a stated justification.
The `gitexec` stdin question is already covered by D4's "added there if it has none".
**Fix:** Settle the import in D7, or carry the question and the fallback into the Notes for the plan writer.

### [LOW] D11's candidate list pins perishable values and the criterion cannot fail first

**Location:** `_lyx/discussion/decision-record.md` D11, Acceptance criteria (repeated integration runs).
**Issue:** Applying the completeness-before-leanness test: the requirement (reproduce under repeated runs, name the dependency, remove it) already sits in the first sentence of D11, so the candidate list is a head start, not a requirement.
Relocation to `support-log.md` would lose it for the plan writer, so it belongs in `## Notes for the plan writer`, without the 30 s and 60 s values.
The criterion passes before and after the fix on a quiet machine, and has no repetition bound.
**Fix:** Move the candidates to the Notes for the plan writer, and require a criterion the found cause makes red to green, with the documented attempt as the fallback.

### [LOW] D8 and D9 change pinned tests and D5's splitter is not stated quote-aware

**Location:** `_lyx/discussion/decision-record.md` D8, D5, Acceptance criteria.
**Issue:** D8 splits "as D5 splits", but `IsReadOnlyCommand` is quote-aware and its test pins separators inside quotes as text, and D5 does not say its splitter is.
D5's `||` rule also changes `IsReadOnlyCommand`, which rejects `||` today.
D8 flips the `TestIsReadOnlyCommand` rows for `git status` and a `lyx` form, and its doc comment says `lyx` and `git` are never read-only; D9 reverses the transcript-shell expiry rows in `internal/shuttleengine/waiting_test.go`.
The record names none of these as rewritten.
**Fix:** State that the shared splitter honours quotes, or that D8 keeps its own, and list the pinned rows that change.

## Rejected

- Fork proposal to relocate D11's candidates to `support-log.md`: fails the completeness-before-leanness test; the list goes to the Notes for the plan writer instead (see the D11 finding).
- Fork proposal that D4's `missing` error name its wrap prefix: an implementation detail the plan writer settles from the code beside it.
- Fork proposal on D8's dispositions when the in-line acceptance is followed by a failed spawn: call-site detail for Plan-Write.
- Fork proposal for D4 empty-stdin, D10 single-test and D13 boundary and rune rows: the rows belong to the card's tests, and the record's criteria already name the behavior.
- Fork speculation that the inherited strand name breaks a fork's built-`lyx` integration tests: unconfirmed, and the D7 Bound already scopes the guard to a mistaken call.

## Focus verdicts

- D9 versus 7462c1dc5: no contradiction for a recovery strand; the gap is the gated run's leftover files, filed as the D9 finding.
- D1 and D2: agree on done cards; the other refusals and the in-flight path are filed.
- D6, D5, D10 and D13 hold up against the code read.
