# Agents change files with the Edit tool, never shell edits

`edit-with-edit-tool` · Task · enhancement, process, stencil, claude · running · Finalize

## Seen (orch, 2026-11-01)

In the three heaviest Webster forks of `run-verbs-shared`, 66 to 90% of the edits were shell edits (`sed -i` 74, 54 and 22 times), often in two steps through a scratch file, at about one edit a turn; the two forks that used the Edit tool made 2.3 to 2.4 edits a turn.
None of them was ever refused by the Edit tool: they read with `sed -n` and wrote with `sed -i` from the start.
The deny-and-steer sentence every agent gets, `steerPythonDeny` in `internal/shellguard/rows.go`, says "edit files with Edit or Write, search and read with grep, awk, cat or sed -n"; nothing stops a shell edit, and the Edit tool takes only a file read with the Read tool.

## Idea

- A file an agent will change is read with the Read tool, a span through offset and limit, and changed with Edit; all of one file's edits go in one message.
- grep and `sed -n` stay for searching and glancing at files the agent does not change.
- The steering sentence says so, and a PreToolUse guard refuses `sed -i`, `perl -i` and a heredoc or redirect into a tracked source file, as the Python guard refuses Python today, with the same steer.
