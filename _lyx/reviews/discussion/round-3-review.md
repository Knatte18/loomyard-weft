---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D2 (renumber journal and its reversal)
    summary: The journal's created-file list is not stated to hold every move's destination name, yet card names carry the slug, so every move lands on a fresh name and a reversal or restore that skips them leaves two files per moved number
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md D2 (journal write before the first apply step; restore-renumber bounds)
    summary: A failed or interrupted journal write has no stated outcome, and restore-renumber has no stated behavior for a journal it cannot parse, so a partial journal blocks both verbs with no way forward
  - id: F3
    severity: LOW
    class: consistency
    location: _lyx/discussion/decision-record.md D2 and Scope (widened loom raw-I/O row)
    summary: The widened raw-I/O row names the card and overview files but not the journal, and its clause nothing is mutated is false for a restore-renumber failure partway through its writes
---

### [MEDIUM] Move destinations are not stated as journaled created files

**Location:** `_lyx/discussion/decision-record.md` D2, the paragraph starting "Before the first apply step they write a renumber journal" and the reversal sentence after it; `restore-renumber`'s fourth bullet.

**Issue:**
`contracts/specs/loom-plan-spec.md` names each card file `NN-<card-slug>.md`, so moving card 8 `08-foo.md` up produces `09-foo.md`, a name that did not exist before the call even though card 9's old file `09-bar.md` did.
Every move destination is therefore a new file name.
D2 journals "the pre-call bytes of every file they will rewrite, rename or delete … and the name of every file they will create", and the reversal "removes every created file and writes back every journaled file under its pre-call name".
Whether a rename's destination counts as a file the verb "creates" is left open, and a plan writer implementing the move as `os.Rename` will read "create" as the new card's file only.
Under that reading the reversal and `restore-renumber` write back `08-foo.md` and leave `09-foo.md` beside `09-bar.md`, so the directory is not byte-identical and fails `index-file-mismatch`, contradicting the acceptance criteria for the injected-failure and restore cases.
The apply order itself is sound: insert moves descending into vacated numbers and remove deletes before moving ascending, so no two files claim one number during apply.

**Fix:**
State in D2 that the journal's created-name list holds every name that did not exist before the call, the destination of each move included, and that the reversal and `restore-renumber` remove each of those names.

### [MEDIUM] Journal write failure and an unparseable journal have no outcome

**Location:** `_lyx/discussion/decision-record.md` D2, the journal-write sentence, the "Both verbs refuse to start while a journal exists" sentence and `restore-renumber`'s bullets.

**Issue:**
The journal is written before the first apply step, but D2 states no outcome for a failure during that write.
The mid-apply reversal covers "a raw I/O failure mid-apply, the journal's removal included", which does not cover the journal's own write.
A short write or a kill during it leaves a partial journal on disk.
Both other verbs then refuse while a journal exists and name `restore-renumber`, and `restore-renumber`'s bounds cover only "no journal" and a journal naming a disallowed path, not a journal that does not decode.
That state has no way forward in any of `refusal-spec.md`'s three admitted forms, even though the plan directory was never touched.

**Fix:**
State that the journal is written to a temporary name and renamed into place, so a journal on disk is always complete, and that a failed journal write removes the temporary file and joins the grouped raw-I/O row with nothing mutated.
Add a `restore-renumber` bound for a journal that does not decode, as its own refusal row with a way forward, and a matching acceptance criterion.

### [LOW] Widened raw-I/O row omits the journal and overclaims "nothing is mutated"

**Location:** `_lyx/discussion/decision-record.md` Scope ("the loom section's raw-I/O row widened to name `rename` and the plan directory's card and overview files") and D2's matching sentence; `contracts/specs/refusal-spec.md` loom raw-I/O row.

**Issue:**
The current row covers `stat`, `mkdir`, `read`, `write` or `remove` of status, seed, lock, park marker or records files, with the way forward "re-run the refused verb; nothing is mutated".
D2 sends the failed journal removal to this row, but the widening names only the card and overview files, so a failed write or removal of the untracked `.lyx` journal still matches no row's wording.
`restore-renumber`'s own raw-I/O failure leaves the journal and is retried by re-running, but by then it may have rewritten some files, so "nothing is mutated" is false for it.
That leaves the row's message and that failure in disagreement.

**Fix:**
Widen the row to name the renumber journal as well.
Either give `restore-renumber`'s mid-restore failure its own transient row whose way forward reads "re-run `lyx loom restore-renumber`; the journal is kept", or state in D2 that it joins the grouped row and qualify the row's clause for it.

## Focus verdicts

1. The reversal-failure halt names `lyx loom restore-renumber`, a `lyx` verb, which is form 1 of `refusal-spec.md`'s `What a way forward is`.
   Because the journal holds the pre-call bytes of every rewritten, renamed or deleted file, keyed by name, it can restore a deleted card and the old `# Card M` headings and `**Commit:**` prefixes.
   Not a defect, apart from the stray move destinations F1 names.
2. D2 states that the pre-call bytes are journaled before the first apply step.
   It places `insert-card`'s new-file write after the descending moves, and `remove-card`'s deletion before the ascending moves, with the Card Index last.
   No two files claim one number during apply.
   The gaps lie in what the journal lists (F1) and in the journal's own write (F2).
3. Scope and D2 both widen the row to name `rename` and the plan directory's card and overview files, so a mid-apply failure on those files has a row.
   The journal file and `restore-renumber`'s partial failure are still uncovered (F3).
