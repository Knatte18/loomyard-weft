# Card 1 — merge-in-progress-kind

**Create:**
- `internal/shedrun#StartMergeInProgressKind`

**Uses:**
- `internal/shedrun#StartNotParkedKind`

**Intent:**
Declare the envelope kind of the new `lyx loom start` refusal over a mid-merge pair, as an exported constant in `internal/shedrun/paths.go` directly below `StartNotParkedKind`.
Its doc comment follows `StartNotParkedKind`'s shape: it names the refusal (a `start` that would spawn or resume a driver over a pair carrying an unfinished merge), says the refusal is NOT retryable (the operator must resolve, conclude or abort the merge first), and says it is exported so an unattended caller tells it apart from the retryable `driver_not_parked` by this one declared value rather than by message text.
Semantic line breaks in the comment.
No consumer yet: card 3 is the only reader.

**Commit:** 1: shedrun: declare the merge_in_progress start refusal kind
