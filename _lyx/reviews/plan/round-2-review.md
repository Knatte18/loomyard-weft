---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/plan/04-loom-card-verbs.md:66
    summary: the lease test cannot observe a release through the real *lock.FileLock as planned, and it never checks that the run-record read happens under the lease
  - id: F2
    severity: LOW
    class: consistency
    location: _lyx/plan/04-loom-card-verbs.md:53
    summary: the stated bounds say the verbs touch only plan-directory files, omitting the journal, its temporary file and the scratch directory they create, and restore's own journal removal
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/plan/02-planparser-renumber.md:69
    summary: reverseRenumber removes every Created name without saying a name not yet created is skipped, which mid-apply reversal and card 3's idempotent re-run both depend on
  - id: F4
    severity: LOW
    class: scope
    location: _lyx/plan/03-planparser-restore.md:25
    summary: readRenumberJournal maps a missing file and an undecodable one but leaves any other read error unmapped, unlike card 2's precheck
---

### [MEDIUM] The lease test cannot see a release, and does not pin the start of the lease

**Location:** `_lyx/plan/04-loom-card-verbs.md:66`, with the `acquireLease` field at `:46`

**Issue:**
The test has `acquireLease` wrap the real `websterengine.AcquireStateMutation` "to record acquisition and release", and the injected `ops` fails the test at any apply step reached while the lease is not held.
`AcquireStateMutation` returns a `*lock.FileLock` (`internal/websterengine/state.go:103`), and the verb releases it by calling `(*FileLock).Release` (`internal/lock/lock.go:76`), a method on a concrete struct.
A wrapper that returns that value cannot intercept the call, so the "release" half of the record does not exist under the type the card implies.
The card never gives `acquireLease` a type, so it leaves open the shape that makes the test work.
With a release the wrapper can see, the test is deterministic: there is no goroutine, and the flag is set and read on the verb's own call path, so a verb that releases before its last apply step fails at that step whatever the scheduling.
The other end is not pinned.
D2 requires the lease "from the run-record read through the last apply step", and the Intent at `:51`–`:52` says the lease comes first.
The test checks only `ops` calls, so a verb that calls `loadState` and runs the `LastBegunCard` check before it acquires the lease passes, which is exactly the begin-batch race D2 bounds.

**Fix:**
Give `renumberDeps.acquireLease` a type whose result the wrapper can intercept, for example `func() (release func() error, err error)`, filled in `newRenumberDeps` from `AcquireStateMutation` and the returned lock's `Release`.
In `TestRenumberVerbs`, also wrap `loadState` so that a call made while the lease is not held fails the test, the same as an apply step.

### [LOW] The verbs' stated bounds omit their writes outside the plan directory

**Location:** `_lyx/plan/04-loom-card-verbs.md:53`–`:55`

**Issue:**
The attack-surface sentence says `insert-card` and `remove-card` "touch only the current generation's card files and the Card Index of `00-overview.md` under `planparser.PlanDir`".
D2 scopes that sentence with "In the plan directory", and the card drops the qualifier.
Under card 2, both verbs also create the plan's scratch directory through `ops.MkdirAll`, write the temporary journal, rename it into place and remove it, and the lease creates webster's scratch directory and lock file.
As written, the bound is false.
The `restore-renumber` sentence has the same gap: it "writes and removes only the base names the journal lists", but it also removes the journal.
Its discard statement at `:55` covers the write-back of `Files` entries, but not that it removes every `Created` name whatever that file holds by then, such as a stub body the agent wrote after the interrupted insert.
For each of the three verbs, the Intent states what it writes and the bound on it, so this is a precision defect, not a missing bound.

**Fix:**
Restore the qualifier ("in the plan directory they touch only …") and add that the verbs also write and remove the renumber journal and its temporary file at `planparser.RenumberJournalPath`.
For `restore-renumber`, add that it removes the journal last and that removing a `Created` name discards whatever that file holds.

### [LOW] The reversal's handling of not-yet-created names is unstated

**Location:** `_lyx/plan/02-planparser-renumber.md:69`; `_lyx/plan/03-planparser-restore.md:30`

**Issue:**
The journal's `Created` list names every destination the call will create, and it is written before the first apply step.
A failure at any apply step before the last move therefore reaches `reverseRenumber` while some `Created` names do not exist yet.
`reverseRenumber` "removes every `Created` name", and the card does not say that an absent name is skipped.
A plain `ops.Remove` of an absent name fails, so most of the `TestRenumber_IOFailureReversed` rows would return `ErrRenumberReversalFailed` instead of `ErrRenumberIOFailed`.
Card 3's claim that `RestoreRenumber` is idempotent rests on the same rule, since a re-run after a partial restore removes names that are already gone.
The planned tests would surface this during implementation, so it is LOW, but the Intent is the contract both cards cite.

**Fix:**
In card 2's reversal sentence, state that removing a `Created` name that does not exist (`fs.ErrNotExist`) is not a failure.
Card 3's idempotency sentence then follows from it.

### [LOW] An unreadable journal has no outcome in restore

**Location:** `_lyx/plan/03-planparser-restore.md:25`

**Issue:**
`readRenumberJournal` treats a missing file as not found and an undecodable one as `ErrRenumberJournalInvalid`.
It names no outcome for any other read error, such as a permission failure.
Card 2's `renumberPrecheck` names this case explicitly (`02-planparser-renumber.md:55`: any other read error is `ErrRenumberIOFailed` with nothing written).
Card 4's `renumberRefusal` therefore has no defined sentinel to map for this path in `restore-renumber`, and no test row reaches it.

**Fix:**
State that any other read error is returned wrapped as `ErrRenumberIOFailed` with nothing written, so it joins the loom raw-I/O row, and add that case as a row of `TestRestoreRenumber`.

## Verdicts on the focus directive's questions

- Journal write path: no defect.
  Card 2's `writeRenumberJournal` creates the journal's parent through `ops.MkdirAll` before the temporary write (`02-planparser-renumber.md:65`), and `TestRenumber_IOFailureReversed` fails that creation as a row.
  Every card-2 test passes a `journalPath` whose parent does not exist yet, and card 4's tests run under a temp anchor whose scratch directories do not exist, which is the case that needs the `MkdirAll`.
  A missing parent on the precheck's read is `fs.ErrNotExist`, so it reads as no journal.
- Attack-surface bounds in card 4: each verb has a stated bound, and restore's discard is stated at `:55`.
  The defect is the precision gap in F2.
- Test economy: no defect.
  The verb-level refusal table reaches each `planparser` refusal again, but what it asserts is `renumberRefusal`'s wording and way forward, which only `loomcli` owns.
  D5 rules out a constructor-only test, so these rows are the only ones that reach each loom-worded message.
  The planparser tests assert byte-identical directories and sentinels, and the verb tests assert exit codes, envelopes and messages, with byte identity asserted only for the begun bound, which only `loomcli` reaches.
  Success cases are one table across the verbs in each package (`TestRenumber`, `TestRenumberVerbs`), not one test per verb.
- Lease test determinism: deterministic for an early release once the release is observable, with no dependence on scheduling.
  F1 covers the missing release hook and the unpinned start of the lease.
