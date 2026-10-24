---
round: 1
focus:
  - "internal/shuttleengine/activity.go `newestTurnEnd`: the loop parses the part before the prompt offset as its own buffer, and each part's parse calls `unreadableFiles.Fail` or `Recover` for the same key; settle whether a parse failure of the pre-offset part, or a Recover from the own part followed by a Fail from the pre-offset part, changes what a run with a readable own turn end or no turn end at all reports or logs compared with the old single parse."
  - "`newestTurnEnd` offset handling: settle whether `min(max(rs.PromptOffset, 0), len)` keeps the old results for a zero, negative and past-the-end offset, and whether an offset that does not fall on an event boundary can make the pre-offset part parse as a torn line."
  - "`readRunActivity`: settle whether the `ownTurnEnd` gate drops the API error in every path that reads a turn end before the offset, and whether `LastActivity` from that reading can never raise a notice beyond making the run read more recently active."
  - "Acceptance criteria against the new `TestReadAgentActivity` row and the doc text in `activity.go` and `doc.go`: settle whether each of the four criteria is covered, with a run holding a turn end past the offset and a run with no turn end at all still asserted by an existing row, and whether the comments follow the repository's one-sentence-per-line rule."
---

Seed focus for round 1.
The diff is small and confined to the provider-neutral reader, so the open questions are the changed parse loop and the API-error gate.
