# Support log — kickstart-read-all-first

## Interview

Autonomous run: no operator answered; every pick below is an auto-pick recorded in the Question ledger.

1. Read the board entry, the related entries (`kickstart-pack-measure`, `kick-start-pack`, `enclose-kickstart`, `weaver`) and commit `113593990`.
   `kickstart-pack-measure`'s "Next audit" waits on this task landing; nothing on the board contradicts it.
2. Read the implementer body and every pack fragment under `contracts/stencils/webster/`, `renderReadRules`, and the `doc.go` pack section.
   Found that the body already puts FRESH-READ first and fills step 1 per mode, so the change fits inside the existing pack-on fragments with no Go or marker change.
3. Settled the read order: turn 1 card files and spans; turn 2 uncovered target and `Uses:` files plus locating unresolved glyphs; their files read whole in the turn after locating; then the per-card loop.
4. Settled that the locate fragment keeps one sentence saying listed lines hold only before any edit, replacing the member-name rule.
5. Lens review, all six forks reported; duplicates merged:
   - premises MEDIUM, locate fragment's "step 1 read the card's listed spans" goes false: fixed in Decision 5 and a criterion, plus a plan-writer note on the test's prefix-only pin.
   - premises LOW, FRESH-READ's "turn at each card's start is that card's fresh read" sentence not named: fixed in Decision 4.
   - premises LOW / pattern-fit LOW, `stencils.go` doc comments in scope though still accurate, and in tension with the frozen registry row: fixed, removed from Scope.
   - premises LOW / board-fit LOW, Decision 7's rationale misstated the audit's baseline: fixed, Decision 7 cut (see leanness).
   - premises LOW, batcher bound premise holds: no change.
   - premises LOW, card-read fragment's 3-space continuation indent: fixed, plan-writer note.
   - goal-scope MEDIUM, Decision 2 extends the board entry unlabelled: fixed, Decision 2 says so.
   - goal-scope LOW, uncovered target files a card creates do not exist yet: fixed, "every existing file".
   - goal-scope LOW / scenarios LOW, dedup has no criterion: fixed.
   - goal-scope LOW, staleness opening of FRESH-READ unpinned: fixed in Decision 1 and a criterion.
   - board-fit LOW, audit's calls-before-first-edit metric rises by construction: fixed, Open risks.
   - board-fit LOW, `weaver` and `kick-start-pack` fit: no change needed.
   - pattern-fit LOW, hash-mismatched deployed stencils are not overwritten: rejected, existing `PATTERN-stencil-ownership` contract, not this task's behaviour.
   - pattern-fit LOW, "docs land in the same commit" unnamed: rejected, `doc.go` is in Scope and criteria, and no CLI or module-table change exists.
   - scenarios MEDIUM, Decision 4 weakens a guard with no bound: fixed, bound stated and open risk widened to stale content.
   - scenarios MEDIUM, empty-pack, all-unresolved and quarry-unavailable cases and `plan:` handles uncovered: fixed, Decision 2 and two criteria.
   - scenarios LOW, "second turn" label loose: fixed, criteria and Decision 2 say "before any edit".
   - leanness LOW, Decisions 6 and 7 repeat Scope: fixed, cut; the byte pin moved to Constraints.
   - leanness LOW, Notes repeat Scope: fixed, replaced with two notes Scope lacks.
   - leanness LOW, untouched-PATTERN constraint bullet: fixed, dropped.
   - leanness LOW, second open risk restates the bound: fixed, now points at Decision 4.
   - leanness LOW, "step 1 no longer reads" overstates given the Intent fallback: fixed, Decision 3 and its criterion say "re-read".

## Rejected alternatives

- **Resolve the pack fresh with quarry before each card (operator's alternative).**
  The fork runs its cards autonomously with no Go step between cards, so a per-card resolve means the fork calls quarry itself per card or Go grows a per-card hook; either is a larger change than the board entry's primary ask, and the operator's stated intent is read-all-first.
- **Put the first-turn rule in a new marker before the card loop.**
  Needs a body marker, a Go fill site and a byte-identity rework for pack-off; the FRESH-READ fragment already sits first and carries the read timing.
- **Fold uncovered-file reads into the first turn.**
  Impossible: plain-path targets and `Uses:` lists are known only after the card files are read.
- **Keep the member-name rule as a fallback.**
  It is the rule the board entry asks to drop; offsets are valid in the first turn and the Edit tool matches by text.

## Review rounds

_No rounds yet._

## Question ledger

- Primary design or operator's alternative? Auto-pick: primary (read all first). Resolved.
- Where does the first-turn rule live? Auto-pick: the pack-on FRESH-READ fragment, no new marker. Resolved.
- Are uncovered-file reads and unresolved-glyph locating batch-wide or per card? Auto-pick: batch-wide, before any edit, labelled in Decision 2 as an extension of the board entry. Resolved.
- Keep any guard against stale listed offsets? Auto-pick: one sentence in the locate fragment, no read rule. Resolved.
- Cap the first turn's read count? Auto-pick: no; listed as an open risk. Resolved.
