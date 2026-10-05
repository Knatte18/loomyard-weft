# Card 7 — The escalation record in the Bouncer run directory

**Edit:**
- `internal/shedadapters/round.go`

**Create:**
- `internal/shedadapters#EscalationCause`
- `internal/shedadapters#writeEscalation`
- `internal/shedadapters#readEscalation`
- `internal/shedadapters/escalation.go`
- `internal/shedadapters/escalation_test.go`

**Intent:**
D5: an escalation leaves two durable files in the Bouncer's run directory, which the wiring commits with the rest of it.

- `round.go` gains `escalationPath(runDir, round)` (`round-<N>-escalation.md`) and `parentNoticePath(runDir, round)` (`round-<N>-parent-notice.md`), beside the other per-round paths.
- `escalation.go` declares `EscalationCause` with `EscalationCircling = "circling"` and `EscalationBudget = "budget"`.
- `writeEscalation` writes the brief file as Go-owned YAML frontmatter (`round`, `cause`) over the rendered brief body, and writes the notice file when `notice` is non-empty.
  The frontmatter is written even when `brief` is empty, so a failed render still leaves the record the `circling` verbs read.
- `readEscalation` returns the cause, the notice text (empty when that file is absent), and whether a record exists for the round.
  A present record is validated through `splitFrontmatter`: the round must match the filename and the cause must be one of the two values, else an error.

Tests in `escalation_test.go`: a write then read round-trips cause, notice and body; an empty brief still yields a readable record; an empty notice writes no notice file; an absent record reads as not existing with no error; a round or cause mismatch is an error.

**ImpactSummary:** One new path pair in `round.go`; no caller until Cards 8 and 10.

**Commit:** 7: The escalation record in the Bouncer run directory
