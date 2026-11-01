```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fixer's change set agrees with the sibling text it touches (fix-consistency lens).

**1. LOW — The `PATTERN-leaf-packages` changes bullet does not state the change it is cited for**
- **Changed text:** `decision-record.md` `## PATTERN changes`, line 226: "`PATTERN-leaf-packages` — changes: Importing into a leaf package: only closed import set."
- **What's wrong:** D9 at line 172 says "PATTERN-leaf-packages admits it (`## PATTERN changes`)", but the bullet only repeats the current overview line from `PATTERN.md:8`, so it records no admission. Only the Constraints line at line 186 says `runproc` gains `internal/lymcwd`. A reader following D9's pointer finds a bullet that changes nothing.
- **Evidence:** `PATTERN.md:8` reads "`PATTERN-leaf-packages` — Importing into a leaf package: only closed import set."

**2. LOW — Two Turn 4 lines in the support log now contradict the record**
- **Changed text:** `support-log.md` `### Round 1`, and the D3 lock bullet and Open risks in the record.
- **What's wrong:**
  - Turn 4 still says "board-scope 2 … kept under Open risks for the parent". That Open risks line is gone and Q11 is resolved. The Round 1 entry does say Q11 is settled, so this one is covered, if loosely.
  - Turn 4 still says "bound-breaker 5 … D3 writes in place; the text is read only at session start and concurrent renders write the same text". F4 replaced that claim with the per-worktree lock in D3, and nothing in the Round 1 entry says the claim no longer holds.
- **Evidence:** support-log Turn 4 lines 31 and 43, against the record's D3 bullet "A render of a worktree holds a per-worktree lock … Two renders of one worktree therefore serialize, and the last one's text stands whole."

**3. LOW — The plan-writer note was not updated for the new `fabricengine.BoltDir`**
- **Changed text:** D9 third bullet: "`fabricengine.BoltDir` (new), built from `BoltDirName`, gives the Bolt checkout's path, and fabric's uses of `BoardDir` that mean the Bolt checkout … move to it".
- **What's wrong:** The note still says "`fabricengine.ConfigDirs` hands configengine the board dir as its `Board` field; under D9 that value is the Bolt checkout, whatever the field is called." Under the fixed D9 that is a use of `BoardDir` meaning the Bolt checkout, so it should name `BoltDir` as its source; as written it reads as though `BoardDir` still gives the Bolt checkout.
- **Evidence:** `decision-record.md` `## Notes for the plan writer`, first bullet.

**Tool calls:**
- `git diff -U0` on the decision record: the context did not show the final text of the sed-changed lines. It returned nothing, which suggests git does not track the file here.
- `grep -n` on the decision record for `SeedWarpExclude`, the D3/D9 criteria and the PATTERN bullets: to read those sed-changed lines in their final form.
