---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: scope
    location: _lyx/plan/10-arbiter-verb.md:54
    summary: The verb's status-read-error refusal has no named test row; only the missing status file and the unreadable envelope are reached.
    fix: In card 10's spawn-path table sentence, add a row for a status file that exists but does not parse (malformed JSON or an invalid state), asserting the refusal, its way forward, nothing spawned and no attempt directory written.
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/10-arbiter-verb.md:58
    summary: A missing status file and an unreadable envelope are classed transient, yet neither clears by itself, and the shed section already classes a missing status file as a correctness halt.
    fix: "In card 10's refusal-spec sentence, class the `arbiter: status or envelope unreadable` row as a correctness halt (as the existing `status file missing` rows are), or split it into a correctness-halt row for the missing status and unreadable envelope and leave only raw I/O on the section's grouped transient row."
  - id: F3
    severity: LOW
    class: design
    location: _lyx/plan/03-shedverbs-arbiter-attempt-flag.md:34
    summary: The flag-without-until-stop refusal is planned in the new TestLoop_ArbiterAttemptConsumedByTheWaiter, though it is a refusal before any step that TestLoop_RefusalsBeforeAnyStep covers once card 3 gives it an argument and hook column.
    fix: "Move the flag-without-`--until-stop` case into card 3's `TestLoop_RefusalsBeforeAnyStep` rows (asserting `kind: bootstrap`, `arbiter_attempt`, no child, hook never called), and leave `TestLoop_ArbiterAttemptConsumedByTheWaiter` covering only the hook-returns-nil path."
  - id: F4
    severity: LOW
    class: consistency
    location: _lyx/plan/14-driver-arbiter-branch.md:15
    summary: Card 14's Bound omits D2's stencil-only bound, so the card that introduces the error-stop edge does not state what a driver running the verb without a fork escalation skips.
    fix: "Extend card 14's `Bound:` sentence with D2's half: a driver that runs the verb at a `producer`, `bootstrap` or reinvoke `interrupted` error without a fork escalation skips the fork's diagnosis and the repair cap, bounded by the spawn budget, the applied-attempt refusal and the arbiter having no repair verbs."
  - id: F5
    severity: LOW
    class: scope
    location: _lyx/plan/10-arbiter-verb.md:43
    summary: A re-run after a verb that died between Start and the marker write reads hand-on cause missing while the arbiter strand still runs unwatched and unstopped, though D1 says a re-run waits on a live arbiter of that attempt.
    fix: In card 10 step 2, before the `otherwise hand-on, cause missing` branch, probe `AttachIfLive` with the attempt's spec (it matches on the verdict output file, not the marker) and wait on a found run as the live-marker branch does; add a re-run-path test row for an attempt with no marker whose strand is live.
---

### [MEDIUM] Status read error refusal unreached by a test

**Location:** `_lyx/plan/10-arbiter-verb.md:54`

**Issue:** Step 1 of card 10 refuses three inputs: a missing status, an unreadable envelope and a status read error.
The spawn-path table names rows for the first two only.
A status file that fails `state.ReadJSONStrict` (malformed JSON) or carries an invalid state is not a raw I/O failure, so the grouped-transient exemption does not cover it, and `PATTERN-refusal-way-forward` asks for a test reaching each refusal.
The focus directive asked for a named test row per refusal; this one has none.
Round 1's F4 named all three refusals, and its fix added rows for only the first two, so this is what remains of that finding.

### [LOW] Missing status and unreadable envelope classed transient

**Location:** `_lyx/plan/10-arbiter-verb.md:58`

**Issue:** `refusal-spec.md` defines transient as a condition that clears by itself or by a named action such as a lock or a retry.
A missing status file never clears on a retry, and the shed section already classes `status file missing` as a correctness halt (`contracts/specs/refusal-spec.md:180`).
Card 10 puts the missing status and the unreadable envelope under one transient row, so the row's class disagrees with the existing rows for the same condition.

### [LOW] Flag-without-until-stop refusal planned in a new test

**Location:** `_lyx/plan/03-shedverbs-arbiter-attempt-flag.md:34`

**Issue:** `TestLoop_RefusalsBeforeAnyStep` (`internal/shedverbs/loop_test.go:587`) is the table of refusals that run no loop, asserting kind, error and no child.
Card 3 already widens it with a column for the step's arguments and the hook, so the flag-without-`--until-stop` refusal is one more row there.
The card instead places it in `TestLoop_ArbiterAttemptConsumedByTheWaiter`, justified as a path "no refusal test covers", which only holds because the row was routed away from the refusal table (`PATTERN-test-economy`).
The hook-returns-nil path is new behavior and fits the new test.
Round 1's F10 proposed keeping this row in the new test, when `TestLoop_RefusalsBeforeAnyStep` had no argument column and ran detached only.
That premise no longer holds, since round 1's own fix gave the table the column, so this finding supersedes that part of F10's proposal.

### [LOW] Card 14's bound omits the stencil-only eligibility half

**Location:** `_lyx/plan/14-driver-arbiter-branch.md:15`

**Issue:** Card 9 says the driver stencil (card 14) bounds the half of D2 that Go cannot check, whether the repair fork escalated.
Card 14's `When:` bullet carries the rule, but its `Bound:` names only Go's eligibility refusal, the budget and the consumption bound.
The attack-surface rule asks the card introducing an edge to state what it can skip and what bounds it; D2 states it (the fork's diagnosis and the repair cap, bounded by budget, the applied-attempt refusal and no repair verbs), and card 14 does not carry it.

### [LOW] Re-run with no marker abandons a live arbiter

**Location:** `_lyx/plan/10-arbiter-verb.md:43`

**Issue:** Step 3 writes the marker only after `Start` returns and the guid is recorded.
A verb killed in that window (the driver's job killed, a compaction that ends the job) leaves an attempt with no verdict and no marker while the arbiter strand runs.
Step 2 then reads `hand-on`, cause `missing`, without probing or stopping the strand, so the driver parks and notifies the parent while the arbiter can still run `circling accept` or `decision add` beside the parent's own handling.
D1 says a re-run "waits on a live arbiter of that attempt"; `AttachIfLive` matches by output-file set (`internal/shuttleengine/attach.go`), so the probe needs no marker.

## Rejected

- Card 2 focus: every refusal `Relaunch` adds has a spec row and a `TestRelaunch` row: the missing status file widens the existing `status file missing` row and has an absent-file row; `done` and `running` each get a new row and a test row.
  `Goto`'s precedence (unknown target, then running, then admission through `CheckGotoTarget`) is stated where `Goto` calls `CheckGotoTarget`, and matches `internal/shedengine/goto.go` today.
- Card 10 focus, deadline: a passed deadline (`now` at or after it) calls `StopStrand` and never `AttachIfLive`, so shuttle's negative-`Timeout` refusal and its zero-means-default reading are never reached; a deadline ahead passes a positive remainder.
  An `AttachIfLive` error reads `hand-on`, cause `died`, after `StopStrand`, and both branches have re-run-path rows.
- Card 12 focus: the bad-number refusal has rows `0`, `-1` and `x`; every other refusal has a row.
  Consumption reads only the verb-written `envelope.json`, never `verdict.yaml`, and a row pins a `verdict.yaml` `resume` under a `hand-on` envelope, so the card's stated bound holds.
- Card 14 focus, wake rule: the re-run is bound to the driver's own last action being the arbiter command whose job ended without an envelope it holds, and excluded on the loop job's exit.
  A recipe-blind driver cannot list attempts on disk, and the verb re-checks the on-disk attempt for the current stop key and spawns nothing for one already arbitrated, so the rule realises D6's condition; not a defect.
- Test economy across cards 2, 9, 10 and 12: card 9 adds no test and is covered by card 10's spawn table; card 12 leaves pause preservation and busy to `TestRelaunch`; card 10's applied-`resume` re-run row and card 12's already-applied row pin different verbs.
  The one overlap is F3.
