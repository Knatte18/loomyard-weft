```yaml
lens: leanness
outcome: reported
```

Task: a leanness review of the decision record and support log, looking for content to move or cut.

1. **MEDIUM — Auto-mode assumptions (polling over Socket Mode; the `@<slug>` prefix).** These are arguments against the options not chosen, and they belong in the support log's Rejected alternatives.
   - The polling bullet argues against Socket Mode ("needs an app-level token…, spreads events across a single app's connections"). The support log already says this under Rejected alternatives.
   - The `@<slug>` bullet argues against `/`, `#` and `>`.
   - The spec rule "Decisions carries Decision + Rationale only… rejected alternatives go to the support log" applies here too: Plan-Write needs only the pick.
   - Fix: shorten each to the pick and the fact that no operator answered. Move the argument into the support log, which partly holds it already. The completeness test passes, because D4 already states the polling mechanism and the prefix.

2. **MEDIUM — Auto-mode assumptions, the Socket Mode fact.** The claim that "Socket Mode spreads events across a single app's connections, so per-hub connections would each miss events" has no cited source.
   - The research only says "up to 10 concurrent connections". It says nothing on whether events are spread across connections or sent to all of them.
   - If the bullet stays in the record, it is an unsourced fact claim. If it moves to the support log (finding 1), it still needs a source or should be marked as the writer's belief.

3. **LOW — Notes for the plan writer.** All three notes repeat `relevant.md`, which already lists `OperatorWaitRows`/`WaitsOnOperator`, `BeamUp` and `resumecontext.go` with the same reasons.
   - The new content is "D8 needs only the two rows it names" and "D3's start belongs on the prime's path only". D8 and D3 already state both.
   - Fix: cut the section. It is optional.

4. **LOW — Constraints, the PATTERN restatements.** The bullets that list `PATTERN-run-process-owner`, `-reap-children`, `-spawn-observability`, `-tolerant-records`, `-durable-vs-ephemeral-state`, `-config-layers`, `-sparse-hub-base`, `-agent-messaging` and `-lym-instructions-in-code` repeat the PATTERN overview, which the plan writer already gets.
   - Keep the parts that add something task-specific: the relay file joins `cmd/lym/daemonwakeups_test.go`'s list, and the D7 rules go in `orch-role.md`.
   - Cut the bare "X governs Y" lines.

5. **LOW — D9 rationale and D3 rationale.** D9's rationale restates `PATTERN-cli-cobra` and `PATTERN-cliwire-sole-wiring`. Fix: drop it, since the decision is a module the pattern already dictates.
   - D3's rationale ("the creel daemon's engine imports no transport") is an argument against hosting the relay in the creel daemon, which is in Rejected alternatives. It can stay as one clause, but it is deliberation.

6. **LOW — Open risks vs Constraints.** The admin-approval fact appears twice: as a Constraint with its source, and as an Open risk. Keep it in one place. The Open risk is enough, with the source moved into it.

7. **LOW (emphasis) — D7, second bullet.** "in plain words without the attach command's need for a terminal" is unclear. It probably means "leave out the attach command", but it reads as if it describes the attach command. Fix: say directly what to drop.

8. **LOW (emphasis) — Goal.** The Goal's second and third sentences restate D4–D8 in summary. That is acceptable as a goal, but the third sentence repeats D6 almost word for word.

I made no further tool calls.
