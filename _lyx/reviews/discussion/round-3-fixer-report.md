# Round 3 fixer report

All five findings fixed in `_lyx/discussion/decision-record.md` and `_lyx/discussion/support-log.md`.
`lyx loom validate-discussion` exits 0 on the result.

## Changed

- F1 (BLOCKING): D12 now adds one self-check verb under `lyx burler` beside the gate entry.
  The verb, the gate entry and the post-gate parse call `ParseReview`; the verb is read-only and has no mode that skips the parse.
  The Scope, Constraints and no-new-verbs lines carve out this one verb, the `PATTERN-gate-self-check-parity` Constraints line states how the pattern is met, and an acceptance line covers the verb.
  The support log's interview item 9 and Q11 record the choice and its reason: the pattern wins over the no-new-verbs line.
- F2 (MEDIUM): D1's matching rule now requires the whole normalized text to equal the box content when the text is shorter than the needle, so a short text inside a longer draft does not match.
  The Bound names the residual case, a draft containing the needle of a text of 48 or more characters, as indistinguishable from the sent text.
  The "draft of other text" acceptance line covers a longer draft containing a short sent text.
- F3 (LOW): D1 uses the 48-character needle `sendVerified` builds.
  The settle is a provider-owned constant through the same injectable sleep, and a new acceptance line asserts the box is read only after the settle that follows an Enter.
- F4 (LOW): D8 defines the turn end being read as the newest turn end among the events read on the tick, and counts two read in one tick as several.
  A new acceptance line covers a boundary with no later turn end.
- F5 (LOW): D9 now says the compaction entries skip the skills step and the retry step and go on to the pointer step.
  D10 says `/reload-plugins` is the first step and is persisted as a step of its own.
  A new acceptance line requires the `doc.go` "The reload sequence" section to describe it and drop the statement that a compaction loses the session's skills.

## Deferred

None.
