## Goal

```markdown
## Seen (orch, 2026-11-01)

In the three heaviest Webster forks of `run-verbs-shared`, 66 to 90% of the edits were shell edits (`sed -i` 74, 54 and 22 times), often in two steps through a scratch file, at about one edit a turn; the two forks that used the Edit tool made 2.3 to 2.4 edits a turn.
None of them was ever refused by the Edit tool: they read with `sed -n` and wrote with `sed -i` from the start.
The deny-and-steer sentence every agent gets, `steerPythonDeny` in `internal/shellguard/rows.go`, says "edit files with Edit or Write, search and read with grep, awk, cat or sed -n"; nothing stops a shell edit, and the Edit tool takes only a file read with the Read tool.

## Idea

- A file an agent will change is read with the Read tool, a span through offset and limit, and changed with Edit; all of one file's edits go in one message.
- grep and `sed -n` stay for searching and glancing at files the agent does not change.
- The steering sentence says so, and a PreToolUse guard refuses `sed -i`, `perl -i` and a heredoc or redirect into a tracked source file, as the Python guard refuses Python today, with the same steer.
```

## Scope

## Decisions

### Added after Discussion (parent, 2026-11-01): Name the harness's bypass-mode text and take sed -n out of the steer

Decision: Claude Code adds a system reminder to every session launched with --dangerously-skip-permissions, which lym's Claude launches carry (claudeengine/command.go), telling the agent it may read with cat, head or sed -n and make small mechanical changes with sed, heredocs or short scripts instead of Read, Edit and Write. The instruction lym gives agents about editing states that, in lym sessions, it overrides that harness text: a file the agent changes is read with Read and changed with Edit or Write, never by a shell edit. The steering sentence (steerPythonDeny and any steer the new guard prints) no longer names sed -n or awk for reading; it names Read for reading a file, and grep for searching.

Rationale: The operator and orch found the harness's bypass-mode reminder telling agents the opposite of the edit rule, the likely main reason forks reach for sed; the steer naming sed -n leads from a sed read to a sed edit (operator, 2026-11-01).

## Constraints

## Auto-mode assumptions

## Open risks

<!-- lym:carry-over Code-Review -->
- Code-Review, round 1: closed by an accepted circling decision; a finding below was addressed by the round's fixer or is still unfixed.
  Review: _lym/reviews/code/round-1-review.md; fixer report: _lym/reviews/code/round-1-fixer-report.md.
  - edit-directive-no-harness-override (class design, severity MEDIUM, open in rounds 1)
  - steer-names-awk-cat-sed-n (class design, severity BLOCKING, open in rounds 1)
<!-- /lym:carry-over Code-Review -->

## Acceptance criteria
