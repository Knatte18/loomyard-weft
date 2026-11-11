---
round: 3
focus:
  - "D6 in decision-record.md: for each reader that uses the known-directory set (LocateSessions callers, ProcessLiveness, tools/tokencount, the session-end usage recorder), does the record state how the accounts, the env module's directory and the hub's worktree-record directories reach it, given shuttleengines.RecordReaders builds engines with empty configengine.Dirs and an engine derives no hub geometry (PATTERN-cwd-resolution)?"
  - "D2 and D6 resume handling in decision-record.md: shuttle runs checkResume before engine.Prepare, so does a resume with no record, or with a recorded directory that is gone, behave as D2 states, and does an acceptance criterion cover it?"
  - "D2 fresh-launch search in decision-record.md: are the sources of the worktree's resumable session ids complete, including the orch's own state record, and does the record say how they reach the claude engine?"
  - "D4 and the process environment: reed spawns its tmux server from os.Environ minus the claude engine's scrub; does the record bound an exported CLAUDE_CONFIG_DIR from reaching the server and later panes?"
  - "D3 counter in decision-record.md: for an account that is re-added, reweighted, or the first to hold a count, does the stated rule keep D3's claims of no catch-up and even spacing, and is every case defined?"
  - "Scope and D6 wording on cost reporting: which component reads transcripts and which reads only stored usage records, and does each rule and acceptance criterion name the right one?"
---

Focus targets the BLOCKING and MEDIUM design areas of the latest round.
