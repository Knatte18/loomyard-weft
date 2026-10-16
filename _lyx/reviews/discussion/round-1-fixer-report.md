# Fixer report — discussion, round 1

## Changes

- F1 (BLOCKING), fixed as proposed.
  Premise confirmed: `internal/loomcli/arbiter.go` `handOn` writes `Verdict: hand-on` with a non-empty `Cause`, a synthetic question and `VerdictPath` set, and `internal/loomengine/arbiter.go` documents `Cause` as empty only for the agent's own verdict.
  D1's signal is now a `hand-on` with an empty `cause` in the attempt's `envelope.json`, and the fork is handed that envelope's path, which sits beside the verdict file the stop report names.
  D1's fallback to the brief's question is dropped.
  D2 lists an arbiter refusal (no envelope) and a `hand-on` carrying a cause as separate fault cases.
  D1's rationale names the empty `cause` as the distinguishing field.
  The second Acceptance criteria bullet names the empty-cause signal, and a new bullet says a `hand-on` with a cause is a fault.
- F2 (MEDIUM), fixed as proposed: D1 gains a Bound paragraph, and the brief's settle condition mirrors the scope and design-wrong limits; the brief's Acceptance criteria bullet states them too.
- F3 (MEDIUM), fixed as proposed: D3 classifies each peer escalation by D1 and D2 first, sends faults to `## Investigating a stop`, puts hand-ons in one joint fork under D1's settle rule and bound with each slug named in the orch note, and starts a late peer's fresh fork only after the first fork's report has arrived and passed the re-check.
- F4 (MEDIUM), fixed as proposed: D7's condition holds through Merge-In and the Merge-In resolution drops the entry; the Open risks bullet names the board note on `reed-add-parentless-placement`; "(running)" is gone from the rationale.
  The Merge-In case is added as its own Acceptance criteria bullet after the last one rather than folded into it, so each bullet states one checkable outcome.
- F5 (LOW), fixed as proposed: D6's Bound now says the file can carry any text into the role file at every delivery, reaches the prime through any landing including an alignment-approved one, and is guarded only by the section's own text.
- F6 (NIT), fixed as proposed: the sentence moved to `support-log.md` Turn 3, and D2's Decision states the rule is kept to one short paragraph.

`support-log.md` also gains a `### Round 1` entry under `## Review rounds` recording these dispositions.

## Disputed

None.

## Deferred

None.
