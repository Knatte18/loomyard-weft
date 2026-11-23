---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: handler
    location: internal/shedadapters/bouncerverify.go#closeVerifiedBlocking
    summary: Go closes any open BLOCKING ledger key the verifier marks closed, including a key whose finding the fixer disputed, deferred or never answered; only the prompt keeps such an entry open, so the entry's rule that the parent is asked when the fixer did not fix a finding rests on the model alone
    fix: "Give each verification entry a required `finding` field naming the review finding id it maps the key to: add it to verificationEntryYAML and to the output block and rules of contracts/stencils/bouncer/bouncer-template-verify.md. Have parseVerification return each closed key with its finding id. In closeVerifiedBlocking, confirm a key only when its entry is `closed` and its finding id is one of the ids fixedBlockingFindings returned; treat any other closed key as open. Add a TestBouncer_FixVerification row whose verifier closes `hard` mapped to a finding the fixer disputed, and assert the round escalates with `hard` open."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/shedadapters/bouncer_verify_test.go#TestBouncer_FixVerification
    summary: No row covers a partial close, so the contract that only the keys still open reach the parent is unpinned for a ledger with more than one open BLOCKING key; every row's ledger holds the single open key `hard`
    fix: "Add a row whose round-2 ledger holds two open BLOCKING keys (for example `hard` and `other`) and whose verification closes only `hard`. Assert the rewritten ledger reads `hard` resolved and `other` open, and that the escalation brief carries `BLOCKING findings open (other)` and not `hard`."
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:code-tests
    location: internal/shedadapters/bouncer_verify_test.go#TestBouncer_FixVerification
    summary: Two failure paths of closeVerifiedBlocking have no row, a verifier spawn that errors and a ledger write that fails, so neither the all-keys-open fallback nor either failure note is pinned
    fix: "Add a row whose RunFn returns an error for the spec with Role bouncerVerifyRole, and assert Awaiting, cause budget and a brief carrying `the check of the findings the fixer reported fixed failed`. Add a row that makes the round-2 ledger unwritable through permkit.Chmod after layout while the verifier closes `hard`, and assert the brief carries `the verified findings could not be closed in the ledger` and the ledger still reads `hard` open."
  - id: F4
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shedadapters/bouncer_verify_test.go#TestBouncer_FixVerification
    summary: The ledger rewrite (markLedgerResolved through renderLedger) is checked only for the status of `hard`; nothing asserts the other entries, rounds, class, severity, fix and prose survive the rewrite that the soft-cap rule, the carry-over and the next judge read
    fix: In the AConfirmedFixClosesTheEntryAndTheRoundAccepts row, compare the whole parsed round-2 ledger after the call against a literal ledgerFile (both entries with their rounds, class and severity, and the fixture prose), not just the `hard` status.
  - id: F5
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shedadapters/bouncer_verify_test.go#TestBouncer_FixVerification
    summary: The selection rules of fixedBlockingFindings have no rows; a fixed-differently disposition, a lower-severity finding reported fixed and a fixer report with `spawned` set are untested
    fix: "Add rows for a `fixed-differently` F1 (one verifier run), for a report carrying `spawned: false` (no verifier run, escalation names `hard`), and add a MEDIUM finding marked fixed to the confirmed-fix row with an assertion in requireVerifierSpec that its id is absent from the prompt."
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/shedadapters/bouncer_verify_test.go#TestBouncer_FixVerification
    summary: The refusals of parseVerification for another round and an empty key, and the re-run of the verifier over an on-disk verification that does not parse, have no row
    fix: Add an onDisk row whose verification names round 1 while the verifier writes closedHard, asserting one verifier run, `hard` resolved and outcome Done; fold the empty-key refusal in as a verifier-written file in another row asserting the failed-check note.
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/shedadapters/bouncerdecision.go#settleDecisionAccept
    summary: The hard-cap halt in settleDecisionAccept archives the judge outputs so a resume re-judges the round, but leaves round-<N>-verification.md, which rejudge now retires because it ruled on the retired ledger's keys
    fix: In settleDecisionAccept's hard-cap branch, archive append(judgeOutputs(b.cfg.RunDir, round), verificationPath(b.cfg.RunDir, round)) instead of judgeOutputs alone, matching rejudge.
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/shedadapters/bouncerverify.go#markLedgerResolved
    summary: markLedgerResolved overwrites the judge's ledger in place with os.WriteFile, which truncates first, so a write that fails part-way leaves a truncated ledger and no copy of the judge's original
    fix: Write the rendered ledger to a temporary sibling in the run directory and rename it over ledgerPath(runDir, round), so a failed write leaves the judge's ledger intact.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: internal/shedadapters/bouncer.go#rejudge
    summary: "The rejudge doc comment still says \"The judge stays the only writer of those files\", but markLedgerResolved now rewrites the round's ledger"
    fix: "Reword the line to say rejudge only archives those files and never writes them, for example \"rejudge itself writes none of those files.\""
  - id: F10
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: pattern/PATTERN-sole-parsers.md
    summary: "The diff adds an on-disk format, the verification file, whose sole parser is parseVerification, and does not add it to the background's format list, which says \"A format another task adds joins this list with its sole parser\""
    fix: "Add a bullet to the \"Formats and their sole parsers\" list naming the Bouncer's verification file and its parser in internal/shedadapters."
  - id: F11
    severity: LOW
    class: decision
    origin: lens:code-scope
    location: internal/shedadapters/bouncerdecision.go#holdAcceptOverOpenEntries
    summary: "The below-window-accept-blocking entry says the run \"should ask the parent instead\"; the code at the base runs another round below the soft-cap window and asks the parent only inside it, and the diff keeps that and reads it as no change, a reading the parent has not recorded"
    fix: Leave the code in place and defer this finding to the run's parent, which records with lym loom decision add either that another round below the window settles the entry or that it orders the parent ask; the fixer changes nothing.
  - id: F12
    severity: NIT
    class: design
    origin: lens:code-correctness
    location: internal/shedadapters/bouncerverify.go#parseVerification
    summary: parseVerification accepts a key listed twice, and a `closed` entry wins over a later `open` entry for the same key, so a self-contradicting verification closes the ledger entry
    fix: In parseVerification, track the keys seen and return an error naming a key that appears in more than one entry.
---

### [MEDIUM] The verifier can close a BLOCKING entry whose finding the fixer did not report fixed

**Location:** `internal/shedadapters/bouncerverify.go#closeVerifiedBlocking`
**Issue:** `blockingEscalationDetail` passes every open BLOCKING ledger key to the verifier, and `closeVerifiedBlocking` closes each one the verification marks `closed`.
Only `fixedBlockingFindings` gates whether a verifier runs, not which keys it may close.
A round with BLOCKING F1 fixed and BLOCKING F2 disputed therefore lists both ledger keys, and nothing in Go keeps the disputed one's key open; the stencil's "An entry that maps to none of the findings above stays open" is the only guard.
The board entry asks that the parent be asked when "the fixer did not fix it", and a model's mapping mistake would land the run past that finding with no parent turn.

### [MEDIUM] No row covers a partial close

**Location:** `internal/shedadapters/bouncer_verify_test.go#TestBouncer_FixVerification`
**Issue:** Every row's ledger has the single open BLOCKING key `hard`, so a regression that sends every key or no key to the parent after a partial close passes.

### [MEDIUM] The spawn-error and ledger-write-failure paths are untested

**Location:** `internal/shedadapters/bouncer_verify_test.go#TestBouncer_FixVerification`
**Issue:** The focus asks whether each verifier failure is an honest escalation.
Reading the code, both paths keep every key open and append a note, which is correct, but only the "writes nothing" failure has a row.

### [LOW] The ledger rewrite's preservation of other entries is unasserted

**Location:** `internal/shedadapters/bouncer_verify_test.go#TestBouncer_FixVerification`
**Issue:** The test checks only `hard`'s status after `markLedgerResolved`; the `alpha` entry, the class, severity and fix fields and the prose could be lost without a failure.

### [LOW] fixedBlockingFindings' selection rules have no rows

**Location:** `internal/shedadapters/bouncer_verify_test.go#TestBouncer_FixVerification`
**Issue:** `fixed-differently`, a non-BLOCKING finding reported fixed, and a report with `spawned` set each take a branch no row reaches.

### [LOW] parseVerification refusals and the on-disk re-run are untested

**Location:** `internal/shedadapters/bouncer_verify_test.go#TestBouncer_FixVerification`
**Issue:** Only the unknown-status refusal and a valid on-disk file are exercised.

### [LOW] The hard-cap halt leaves the verification beside a ledger it retires

**Location:** `internal/shedadapters/bouncerdecision.go#settleDecisionAccept`
**Issue:** `rejudge` archives the verification with the judge outputs, but the other site that archives judge outputs for a re-judge does not.
The round's escalation record keeps `closeVerifiedBlocking` from reading the file today, so this is a consistency gap rather than a live misuse.

### [LOW] The ledger rewrite is not atomic

**Location:** `internal/shedadapters/bouncerverify.go#markLedgerResolved`
**Issue:** `os.WriteFile` truncates before writing, so a failed write leaves an unparseable ledger, after which the round escalates on an unreadable ledger and the judge's record is gone.

### [LOW] rejudge's doc claims the judge is the only writer of its outputs

**Location:** `internal/shedadapters/bouncer.go#rejudge`
**Issue:** Go now rewrites the ledger, so the sentence is false.

### [LOW] The verification format is missing from the sole-parsers list

**Location:** `pattern/PATTERN-sole-parsers.md`
**Issue:** The background asks a new format to join its list.
The Bouncer's ledger, verdict and focus formats are absent too, which is why this stays LOW.

### [LOW] below-window-accept-blocking is read as needing no change, without a recorded parent call

**Location:** `internal/shedadapters/bouncerdecision.go#holdAcceptOverOpenEntries`
**Issue:** The entry's aim, that no run lands past an open BLOCKING entry, holds at the base: below the window the accept is held and another round runs.
Its literal remedy, asking the parent, is not built, and only the parent can accept that reading.

### [NIT] parseVerification accepts a duplicated key

**Location:** `internal/shedadapters/bouncerverify.go#parseVerification`
**Issue:** A ruling listing a key both `closed` and `open` closes it instead of being refused as malformed.

## Rejected

- The verifier pass is recorded in the usage record's judge list (code-correctness finding 3, code-tests NIT): the verifier runs on the judge's model and config role by design, and the usage record has only a seed and a judge list, so this is attribution, not a defect.
- The verify stencil's `git log --oneline` does not narrow the log to this round's fix commits, and finding ids recur across rounds (handler): the verifier rules on the code as it stands against the finding, so an earlier round's commit with the same id does not change what it checks.
- The verify stencil is read with stencilstore.Read rather than ReadForEngine (handler): it holds no engine-block include, and every other Bouncer stencil reads the same way.

Focus directives, each reached: the verifier closes an entry only on a parsed `closed` ruling for a listed key, after a fixed or fixed-differently BLOCKING disposition and a target not recorded as unchanged (F1 covers the gap in which keys it may close); every failure path escalates with all keys open and a note (F3 covers its tests); of the folded entries the diff lands none and changes nothing they do not license, with the three no-change readings borne out by the cited tests and code and the fourth deferred as F11; the recipe, spec, doc, parity and stencil changes match the new step, and the stencil's output block matches parseVerification; the tests assert literal values and do not duplicate TestBouncer_AutoAccept, with the gaps in F2 to F6.
