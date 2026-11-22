```yaml
lens: leanness
outcome: reported
```

Task: a leanness review of the decision record and support log, looking for content to cut or move.

1. LOW, D4 Rationale: this repeats the Constraints bullets. Its first sentence restates what Constraints already says: cancel without `--remove` keeps the pair, makes no tags and blocks a re-run, and `--remove` archives then deletes both branches. The fix is to keep it once, in Constraints, and cut the D4 rationale to the release-ownership sentence. Evidence: the Constraints bullets "`lym batten cancel` sets a run-form status back to `draft`… with `--remove` it archives each branch tip…" and "Cancel without `--remove` keeps the pair and makes no archive tags…".

2. LOW, Goal: the last sentence is deliberation that belongs in the support log. "No verb re-parents a run; the study … stays the record of why" says why the verb was rejected. Scope "Out" and D1 already say that no verb is added, and the support log's Rejected alternatives holds the re-parent verb. Cut the sentence from the Goal.

3. LOW, Scope "Out": the bullet "This task's own board entry" carries no requirement. No decision touches the entry, and the board lock already enforces this. Cut it.

4. LOW, Notes for the plan writer: the `orch-autonomy-mode` overlap note is a text-only overlap of files. The rubric and the board-scope lens treat that as no finding, and it gives the plan writer nothing to act on. Cut it. The second note, about glossary and role tests still passing, is generic too, and it could go as well.

5. LOW, Constraints: the `PATTERN-github-auth` bullet carries no constraint. "The procedure opens or closes no pull request; cancel's teardown does what it does today" adds nothing beyond D1's "built on existing verbs". Cut it, or keep only the PATTERN-compliance clause if the gate wants each touched entry named.

6. MEDIUM, D2 (also seen outside leanness): "Merriam" is a nickname in a file. CLAUDE.local.md says to "Keep the formal names in code, files, verbs and specs", and the record feeds role text. Write "the Webster master session" in D2.

7. LOW, support log Turn 1: "Facts settled" repeats the record's Constraints almost word for word. That is acceptable in the support log, but the support log could point at Constraints instead of restating them. This is optional, since the plan writer never reads the support log.

I found no content that has to move out of the record for completeness. Every decision's bound and the open risk carry requirements the plan writer needs.

I made no further tool calls.
