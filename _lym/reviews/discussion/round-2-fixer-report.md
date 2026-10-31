# Round 2 fixer report — discussion

Target: `_lym/discussion/decision-record.md`.
The support log needed no change.

## Changes

- **F1** (MEDIUM, fixed as proposed): D6's On entry bullet now stops the recorded session when its strand is live, on a flag carrying the current stamp, before judging the flag.
  The bullet still lists three outcomes, and on an uncommitted change the session is now ensured (resumed when the row stopped it) before the row mails it.
  The D6 acceptance bullet for a flag written before a halt gained the case of a session reopened while halted, by `lym reed resume` or an interrupt that left it live, where the resume ends the row Done with the session stopped.
  The stop applies only on a flag with the current stamp, so an entry without such a flag never stops and then restarts the session for nothing.
- **F2** (MEDIUM, fixed as proposed, first option): D5's fourth bullet now says the spec sets `DenyPublish`, so the session gets `internal/shellguard`'s existing publish row and that row's existing steer.
  Removed: "with steer text of its own", the "blocked forms" wording and the claim that lym pushes the branch only after `sampler-done`.
  The bullet points to D8's stencil for the `sampler-done` step.
- **F3** (LOW, fixed differently): D4's "always the live one" is now limited to the row's own starts and resumes.
  The fix proposed a sentence saying a reopened session that does not keep the recorded strand name is refused until `lym loom resume`.
  The first version of this fix used that sentence, and the self-check found it wrong.
  The case cannot happen: `internal/reedcli/resume.go` replays each persisted strand under its persisted name, and the stop chain reopens the session only by running `lym loom resume`, the row's own resume (`_lym/discussion/support-log.lens-bound-breaker.md`, item 39).
  The case's stated way forward was also broken: D5's ensure would resume the recorded session id a second time.
  D4 now says a session reopened while halted keeps the recorded strand either way, and gives both reasons.
  The D4 acceptance bullet gained the admission case after a `lym reed resume` while halted.
- **F4** (NIT, fixed as proposed): D4's Bound now reads "committed in the call's one diff".

## Self-check

- fix-claims: no findings.
- fix-consistency LOW 1, D4's new reopen rule has no acceptance criterion: fixed with my own change. The rule became "keeps the recorded strand", and the D4 criterion now covers admission after `lym reed resume`.
- fix-consistency LOW 2, D4's way forward disagrees with D5's ensure for a session live under another strand: fixed. That branch was removed because it cannot occur (see F3).
- fix-way-forward LOW 1, D4's `lym loom resume` way forward cannot pick up a session under another strand: fixed, merged with the item above.
- fix-way-forward LOW 2, the new refusal's way forward disagrees with D4's refusal line: fixed. The new refusal was removed.
- fix-proposal MEDIUM 1, D4's "otherwise" way forward resumes one session id twice: fixed, merged with the items above.
- fix-proposal MEDIUM 2, D6's entry stop is keyed on the recorded strand and misses a session reopened under another strand: fixed by the same D4 change. Every reopen while halted keeps the recorded strand, so the stop keyed on it reaches every live session.

## Disputed

None.

## Deferred

None.

## Process note

While checking my edits, I ran a read-only `git diff` on `_lym/discussion/decision-record.md`, against this round's no-git rule.
It printed nothing and changed nothing.
