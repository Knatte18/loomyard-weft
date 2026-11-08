```yaml
lens: leanness
outcome: reported
```

Lens: leanness. I looked for content in the decision record to cut or move, and checked whether anything a move would drop is something the plan writer needs.

1. **MEDIUM — `## PATTERN changes`, `## Constraints`, `## Notes for the plan writer`: the `PATTERN-agent-addressing` change is stated three times.**
   - The `changes:` bullet repeats the current overview line word for word, so it prescribes no edit.
   - The Constraints bullet "The background's Slack boundary rule now names the three tokens" repeats what the first Notes bullet says.
   - Fix: keep the full new background rule once, in the Notes bullet. Point the Constraints line at it and keep the PATTERN bullet, since an entry change cannot go undeclared. Give the PATTERN bullet a short "background rule only" mention instead of repeating that in Constraints.

2. **MEDIUM — the Rationale of D1, D9 and D11 argues against alternatives the record did not choose.**
   - D11: "it adds less than a hand-written frame codec would to maintain".
   - D9: "One presence per orch would have them set it once per beam".
   - D1: "needs no new process kind and no election protocol".
   - The spec's compaction rule keeps rejected alternatives out of `## Decisions`, and the support log's `## Rejected alternatives` already holds each of these.
   - Fix: cut those clauses and keep the positive reasons.

3. **MEDIUM — `## Open risks`: two of the three bullets are not risks for the parent.**
   - The `slack-login-prompt` bullet is a text overlap of one verb and file. D3 already settles it ("keeps the reading `slack-login-prompt` gives it"), and the board-scope rule treats a text-only overlap as no finding. Rebasing is routine.
   - The acknowledgement-deadline bullet says it is "not expected to bind", so it raises nothing.
   - Fix: cut both, or move the deadline fact into Constraints' Slack facts as "no documented Socket Mode ack deadline". Keep only the archived-name bullet.

4. **LOW — the `slack-login-prompt` boundary is stated three times: Scope Out, D3 Rationale and Open risks.**
   - Fix: keep the Scope Out line and the D3 clause.

5. **LOW — `## Constraints` repeats facts that D1 and D3 already state.**
   - The last bullet's Slack facts restate the D1 Rationale (one connection per payload) and the D3 Rationale (unarchive refuses bot tokens), each with its source already cited.
   - Fix: keep only the facts no Decision cites: the ten-connection cap, ack-or-retry, the refresh `disconnect`, the name grammar and `chat:write.customize`. Drop the two duplicates.

6. **LOW — `## Auto-mode assumptions` mostly restates Decisions.**
   - "prime never archived", "presence per hub", "threads forwarded", "files named", "no icon" and "channels public" are D6, D9, D7, D8 and D5 almost word for word.
   - The section is required, but each line can name its pick and its D-id in a few words rather than restate it.
   - Only the shortname-as-repository reading and the user token required everywhere carry an assumption that no Decision states as one.

7. **LOW — the third bullet of `## Notes for the plan writer` duplicates D8.**
   - D8 already says "`agentname.TaskSlug` and the existing program parse give the slug", so the bullet adds nothing. Cut it.

8. **LOW — the D9 Rationale sentence about how `slack-presence-auto` "fits a channel per orch unchanged" is forward-looking narrative about another note.**
   - The plan writer acts on nothing in it. Move it to the support log, or cut it.

I found no content that belongs outside Discussion entirely. Each relocation above keeps every requirement in the decision record.

Tool calls made: none.
