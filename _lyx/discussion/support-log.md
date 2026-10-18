## Interview

A darn run holds no interview: the writer read the board entry and its scope section, the parallel-batches study and `internal/planparser`, and made its own calls.

## Rejected alternatives

- Exact column equality only: misses a card editing `pkg#T.M` after one creating `pkg#T`, the overlap the study's symbol level counts.
- The study's rule alone, with no edge for an earlier read of a later change: would show two cards as parallel while one reads what the other rewrites.
- Drawing every direct edge: dense plans become unreadable, and reachability is the same.
- A Go graph package: the board entry rules it out; DOT is plain text.
- Defaulting the plan directory to the current worktree's plan: needs cwd resolution through `internal/lyxcwd` for little gain; the plan directory is a required argument.
- A new `internal/planparser` export for column keys: `RefMember` already gives the canonical member form.

## Review rounds

_No rounds yet._

## Question ledger

- Should use-use pairs get an edge? Self-pick: no, two reads never conflict. Resolved.
- Which direction does an edge run? Self-pick: earlier card to later card, the order webster runs cards. Resolved.
- Where does output go? Self-pick: `-out` directory, default `.scratch/plandag`, fixed file names. Resolved.
