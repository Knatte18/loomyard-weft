---
round: 3
focus:
  - "Pin sequencing in decision-record.md D5: tmux exits the server when exit-unattached is on and no client is connected, and reed runs each tmux command as its own client. Settle whether the decision places the post-claim pins (global destroy-unattached off, server exit-unattached off) in the same tmux invocation as the conditional source, so a file setting exit-unattached on cannot end the server first, and whether the third acceptance criterion can then pass."
  - "Spawn-to-pin window in D5 step 1 and D8: concurrent boots are not serialized. Settle whether a boot's session-local destroy-unattached off and its @lyx_reed_session tag are applied in the same command queue that creates the session, so another boot's global destroy-unattached on or a concurrent last-worktree down cannot kill the session or the server under it, and whether an acceptance criterion exercises two concurrent boots against a file setting destroy-unattached on."
  - "Candidate rule in D3 step 1: Down and EndSessionByName kill the server when no session remains and are not serialized with another worktree's boot. Settle whether a boot whose observed server dies before its own spawn still sources the file, or whether the decision states the bound of the resulting file-less server and what clears it."
  - "Claim command string in D2 and D3 step 2: the conditional claim nests source-file and the path inside a command string that tmux parses a second time. Settle whether the decision refuses whitespace and tmux parser metacharacters in the path or states that the path is quoted for that parse, and whether the path acceptance criterion has a matching row."
  - "Session-listing surfaces in D8: reed reads that list or cycle the server's sessions include discovery, reaping, status, the next and previous session switch, and the status bar session loop. Settle whether D8 is a rule over every such read, or names the surfaces that deliberately show every session with a stated bound."
---

Each entry names where to look and the question to settle; none pre-states a verdict.
