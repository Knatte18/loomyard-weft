# Board entry `turn-audit`: Audit where agent turns go and which could be merged

Calls per turn, mergeable serial turns and the read-then-edit-then-test pattern per role, from transcripts only

## Question

A session's cost grows with its turns times its context: every turn reads the whole context again from cache, and every early turn grows the context the later turns read (`docs/research/token-audit-2026-10-22`: the Plan writer averages 81 turns at 223k, a Webster fork 62 at 179k, the darn writer 58 at 124k, the Plan reviewer 48 at 123k).
This audit finds where turns go and which of them could have been merged, as the baseline for stencil and system-prompt changes that cut turns.
Agents keep reading the source themselves: no change proposed from this audit hands an agent pre-read or digested source (operator, 2026-10-28).

Per role (Plan writer and its lens forks, Discussion writer, Webster implementer forks, darn writer, Discussion, Plan and Code reviewers and fixers, merge fixer):

1. **Calls per turn.** Turns per session, tool calls per assistant turn, and the share of turns with a single tool call.
2. **Mergeable turns.** Runs of consecutive single-call turns whose calls do not depend on each other's results, such as reads, greps, `lym quarry` calls and lint runs back to back; their count and the share of the session's cost they carry, since a later turn costs more than an early one.
3. **Read, then edit, then test.** The pattern from the r/ClaudeCode thread in note `claude-md-thread-ideas`: gather every file needed before the first edit, make all edits in as few turns as possible, then test once. Per session: reads after the first edit, switches between reading and editing, edits per turn, edits that touch a file edited before in the same session, and gate, test and lint calls per session.
4. **Kinds of serial calls.** Which tools and commands make up the mergeable turns (Read, `sed -n`, `grep`, `lym quarry`, `lym loom validate-plan`, lint, `lym gate test`), so a fix can name them.
5. **Recommendations.** Per role, the concrete stencil wording or system-prompt sentence (appended by shuttle with `--append-system-prompt`) most likely to cut turns, and an estimate of the turns and cost each saves; marked as recommendations, not decisions.

## Scope and cap

- Sessions: the Claude Code transcripts of loom and darn runs landed on Neve since 2026-10-20, found as `tools/tokencount` finds them; runs from OSL-1046 are left out.
- No agent or model run beyond the darn writer itself: no bench, no replay, no grading agents.
- Output: `docs/research/turn-audit-2026-10-28/report.md`, with the sample size beside every number, and the scripts that produced the tables beside it.
