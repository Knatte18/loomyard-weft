# Card 11 — delete-fix-start-type

**Delete:**
- `internal/burlerengine#FixStart`
- `internal/burlerengine#FixStartParallel`
- `internal/burlerengine#FixStartAfterReview`

**Intent:**
Delete burlerengine's `FixStart` type and its `FixStartParallel` and `FixStartAfterReview` constants (decision D5).
Card 7 removed every use of them and card 8 rewrote the round without them, so nothing references them by now; this card removes only their declarations in `profile.go`.
It is a pure deletion and relies on the existing tests.

**ImpactSummary:** Deletes three unused declarations in burlerengine's profile.go; no caller remains after cards 7 and 8.
