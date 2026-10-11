# Fixer report — discussion round 1

Every finding's premise was checked against the code before fixing:
the shared directive lines in `contracts/stencils/seat/seat-directive-chair.md` and `seat-directive-advisor.md`;
`KeepPane: !isChair` in `internal/seatengine/prompt.go`;
the reviewer wait-error case in `decideRound` (`internal/burlerengine/handoff.go`), which returns `burler: shuttle run: <err>`;
the stale-marker removal before either half starts in `internal/burlerengine/engine.go`;
and `AuditForks` running ahead of the `KeepPane` cleanup in `internal/shuttleengine/wait.go`'s finish.
All seven premises hold.
`lyx loom validate-discussion` exits 0 after the fixes.

## Changes

- **F1 (BLOCKING), fixed differently.**
  D8 now says the Burler seat stencils include neither shared seat directive, names the conflicting lines, and has each Burler stencil state its own channel rules; the shared directives' header comments name the Burler stencils as the channel wording's other home.
  D8's bound is split into a Go-enforced part (skipped-handoff error, post-fix byte comparison) and a stencil-only part.
  D7 now says the chair waits inside its turn and never ends it to await the up line.
  Scope's stencil bullet and the PATTERN-producer-pointer-rule constraint now match, and an Acceptance criterion pins the stencil content.
  Not taken: the proposed option of including the shared directives with Burler text naming which lines don't apply.
  It still puts two contradictory rule sets in one prompt and relies on the agent to resolve them, which is the defect the finding names.
  A Burler directive variant would duplicate the shared block's wording.
  The rejected options are recorded in the support log's Rejected alternatives.
- **F2 (BLOCKING), fixed as proposed.**
  D8 states that a reply only explains a finding the accepted review holds, that the fixer neither acts on nor fixes anything a reply raises beyond the accepted review, and that the reviewer writes no file after its review.
  The bound on the reviewer's writes is stencil-only, and the residual risk (a kept reviewer with edit tools writing a target file or the fixer-report, or a reply carrying a new finding) is under Open risks.
- **F3 (MEDIUM), fixed as proposed.**
  D3 now ends the step when a required advisor ends done with its gate failed, and its bound says every way a required advisor ends without a successful `Accept` ends the step.
  New Acceptance criterion for the case.
- **F4 (MEDIUM), fixed as proposed.**
  D4 keeps today's rule: a reviewer wait error is returned wrapped, not as a nil-error outcome.
  `SeatResult` also carries the wait error, and the D4 Acceptance criterion lists the wait-error case.
- **F5 (LOW), fixed as proposed.**
  D5 states the stale marker is removed before the chair starts, and why chair-first makes it the guard.
  D3 states the seat gate is valid only on an advisor and `Table.Validate` refuses it on the chair.
  The marker Acceptance criterion now cites D5.
- **F6 (LOW), fixed as proposed.**
  New llm-tier Acceptance criterion: an advisor turn started by a chair's SendMessage reads as mid-turn by the session-state fold.
  The Open risk now points at that criterion and carries no "must".
- **F7 (NIT), fixed as proposed.**
  Dropped the KeepPane fork-audit risk; the existing cluster-round Acceptance criterion already pins the exact-N audit.

## Disputed

None.

## Deferred

None.
