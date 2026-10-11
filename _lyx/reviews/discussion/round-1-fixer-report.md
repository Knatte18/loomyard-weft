# Fixer report — discussion round 1

Every finding's premise held against the code; all seven are fixed.

## Changes

- **F1** (fixed as proposed, using existing bounds): D5 gains a Bound paragraph.
  A fork that never sends a final notice is bounded by the run deadline and the liveness check (`internal/websterengine/doc.go:506`).
  A final notice misread as interim, or an interim notice whose fork no longer counts as outstanding, ends the run with nothing done under Master's safe-turn-end rule (`contracts/stencils/webster/webster-template-master.md:9`), so the run ends instead of hanging and its outcome shows the unrecorded batch.
  These existing bounds hold, so no new one was decided.
- **F2** (fixed as proposed): D5 extends the interim-notice line to the fixer fork's step 4 under "A gate failure" (`webster-template-master.md:168`).
  D3 gives `webster-body-verify-fix.md` the same no-backgrounding rule.
  The verify-fix and Master criteria cover both.
- **F3** (fixed as proposed): new D9 has claudeengine's turn-end activity reading take the newest write across the main transcript and the fork transcripts under `subagents/`, and read a fork holding a tool call open as active.
  Grounded in `internal/shuttleengine/activity.go` (`readRunActivity` reads only the main transcript's mtime and the newest Stop line) and `claudeengine/audit.go:50` (forks write under `<session>/subagents/`).
  The third Open risk is removed; Scope, Constraints (doc line), Auto-mode assumptions, the criteria, the support log's Rejected alternatives and ledger Q9 carry it.
- **F4** (fixed as proposed): `CLAUDE.md`'s push rule is in Scope, D7 and the Notes list; D7 and the grep criterion now say "outside `_lyx/`" instead of the undefined "production text".
- **F5** (fixed as proposed): D6 says the orch copies the gate rule, timeout value included, from its own system prompt into a non-fork agent's brief, and that this runtime copy is the one sanctioned restatement; the stencil names the source and never spells the rule.
- **F6** (fixed as proposed): D1's Bound names both outcomes at the ceiling, ended and backgrounded, and points the backgrounded one at D5.
- **F7** (fixed differently from one option): D2 keeps its claim as the one statement and now names the `cd` and quick-check rules among what stencils point at; D3 and D4 point at the gate rule instead of restating them.
  The criteria cover the verify-fix `cd` pointer and the worktree-relative path rule for both bodies.
  I did not narrow D2's claim to the timeout value, the Fix line's other option, since pointing keeps one statement and removes the divergence between "never `cd`" and the steer's narrower rule.

## Disputed

None.

## Deferred

None.
