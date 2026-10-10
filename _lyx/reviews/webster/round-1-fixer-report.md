# Webster round 1 — fixer report

I checked all three findings against the code, and each premise holds.
Each fix is its own commit, and `lyx gate test` passed on the touched package before every commit.

## Fixed

- **F1** (`1ba07d045`): the `TestIsReadOnlyCommand` row "git config get with a file" is now `git config --get --file=f k`.
  That is three words after `config`, so the count bound passes and the row reaches the `-`-prefixed word check in `readOnlyGitSegment`.
- **F2** (`3a343035c`): the class loops of `TestCheckFork` and `TestCheckParent` now assert that each violation's class renders a `pathlessEntry` that `pathlessEntryClass` recognises with that class.
  Between them, the two tables exercise all seven `AuditViolationClass` constants.
  A class the audit emits but `pathlessViolationClasses` omits now fails there.
  The list stays beside its writer in `recordbatch.go`, as Card 1 places it.
- **F3** (`5db5e4c1a`): `readOnlyLyxSegment` drops the `word.glob ||` operand.
  `holdsGlobCharacter` on the dequoted text is true wherever `word.glob` is.

## Disputed

None.

## Deferred

None.
