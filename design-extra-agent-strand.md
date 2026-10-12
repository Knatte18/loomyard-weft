# Open an extra named agent strand through shuttle

`extra-agent-strand` · Note · feature, shuttle

The operator wants extra Claude strands beside the orch in the prime, such as one for exploring the repo, added by the orch on request (operator, 2026-10-06).

Tried 2026-10-06: `lyx reed add --name explore --cmd claude` runs a bare `claude`, so the pane title is `ly:explore` but the Claude session gets no `--name`, and reed records no session id, so the watchdog's session-name repair (`internal/reedengine/namerepair.go`) skips it.

Every agent in lyx runs through shuttle (operator), so this needs a shuttle engine extension: open a named LLM session in a strand with no turn and no prompt, started with the strand name and its session id recorded, so the watchdog keeps the name across a compact.
The verb is not in reed, which shuttle depends on.
Shuttle now has a CLI of its own (`internal/shuttlecli`: `run`, `send`, `await`, `interrupt`, `state`), so the verb could live there.

Open: where such a strand goes in the reed layout relative to the orch, and how several share the space.
