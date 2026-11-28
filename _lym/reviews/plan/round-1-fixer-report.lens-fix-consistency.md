Lens: fix-consistency, checking the plan edits against sibling cards, criteria and cross-references.

**MEDIUM: card 12's `plantRunTransition` move edits another concern's file**
- **Changed text:** card 12 (`12-board-test-moves.md`), the bullet "`plantRunTransition` moves from `codetip_integration_test.go` to `codetipsettle_integration_test.go` …".
- **Summary:** the move contradicts two things.
  - Card 4 says moving `originStatusAtTip` into `handover.go` "is the one edit of another concern's file this part makes".
  - The decision record's review criterion says "each part's diff changes in `internal/fabricengine` only its own concern's files besides the executor and shared types, in D9's order (D9)". `codetipsettle.go` is the code-tip settle, which the record lists with the takeover write as a separate write path.
- **Evidence:** `_lym/discussion/decision-record.md:384` and `:219`; card 4's line "Only the function moves and handover.go's own code is untouched: it is the one edit of another concern's file this part makes".
- **Fix:** either card 4's sentence and card 12 must say this is a second such edit and give the reason, or the helper must stay in a file of this concern.

**LOW: the overview's sequential-seam decision still lists only cards 4, 9 and 11**
- **Changed text:** cards 7 and 8 each gained a `TestInstallRequiredSeams` row that unsets the run-records seam through `fabricengine.SetRunRecordPaths(nil)`.
- **Summary:** the Shared Decision "required seams are set only through their setters, sequentially" still reads "Applies to: cards 4, 9, 11", so its twin was not updated.
- **Evidence:** `00-overview.md`, the decision's "Applies to" line.

**LOW: card 3 still sends every adapter's check to card 4's test**
- **Changed text:** card 4 now says "The run-lock adapter's check is card 11's". Cards 2 and 3 add a `Recorded` adapter.
- **Summary:** card 3's Tests bullet still reads "The installer's adapter test against the real shed module is card 4's `TestInstallRequiredSeams`".
  - The run-lock check now lands in card 11.
  - No card's `TestInstallRequiredSeams` checks the new `Recorded` adapter, though card 4 still claims "each adapter names what shed writes".
- **Evidence:** `03-required-seam-installer.md`, its Tests section; card 4's `TestInstallRequiredSeams` bullet, which checks only the paths and codec `LastMachine`.

**LOW: `codetip_integration_test.go`'s file header goes stale**
- **Changed text:** card 12's `plantRunTransition` move.
- **Summary:** the file header says "It also holds the helper that plants one, which the owner-guard rows of the pair push share". That stops being true once the helper moves, and no card says the header is rewritten.
- **Evidence:** `internal/fabricengine/codetip_integration_test.go:2`.

Tool calls:
- I grepped and read `_lym/discussion/decision-record.md` around lines 210–222 and 384 to find which concern `codetipsettle` belongs to and what the per-part file rule says. My context held only the record's added entries and its constraints.
