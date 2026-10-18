# Support log — tokencount reports cost per segment

## Interview

A darn run holds no interview: the writer read the board entry and the code and made each call itself.

## Rejected alternatives

- An explicit role-to-segment table: rejected for the prefix rule, which needs no edit when a segment gains a role.
- Summing sub-row times for a segment's time: rejected, since overlapping sessions would count twice.
- Keeping the weight-based share: rejected, since the report ranks by cost and the weight column moves behind `-detail`.
- Placing "Most expensive parts" above the title and header lines: rejected, since the header names the counted runs the list is about.
- A sub-row for the orch average per run: rejected, since sub-rows must sum to their segment; the average sits in the segment row's name instead.

## Review rounds

_No rounds yet._

## Question ledger

- Which segment holds `recovery`, `arbiter`, `conflict`, `describe` and `friction`? Self-picked: `recovery` is webster's recovery strand and joins Webster; the rest join Other.
- Is the sessions column a token column? Self-picked: it moves behind `-detail`, so the default tables hold cost, share, time and models only.
- What is Alignment's writer called? Self-picked: `session`, since the alignment agent judges rather than writes.
- Where does the orch average per run show? Self-picked: in the "All runs" Orch segment row's name, and in the Cost per run table's average row.
