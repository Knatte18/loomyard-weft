---
round: 2
focus:
  - "D1 in decision-record.md: settle where the coverage read sits relative to shuttle's idle wait before typing (sendVerified calls awaitIdleSession then typeVerified in internal/shuttleengine/run.go), given a tender pause can land while a send waits for idle; check the text and acceptance criteria against the declared PATTERN-pause-holds-prompts."
  - "D6 and D7 in decision-record.md: settle which identifier the orch's session record carries, comparing against the strand identifier orchengine's watcher writes (State.Strand in internal/orchengine/state.go) and the matches in internal/orchengine/pause.go."
  - "D1 and D4 in decision-record.md: settle what lyx orch start and a Verify entered under a covering pause do when SetPause arms waitWhileCovered or when verifytree.Verify starts its command, and whether the text and acceptance criteria state it."
  - "D4 in decision-record.md: settle whether the stop's SIGKILL escalation follows from verifyrun's exec settings (cmd.Cancel, cmd.WaitDelay in internal/verifyrun and internal/proc/procgroup_linux.go) and which verifyrun.Run callers (batten deploy and Command rows, cardverify) the changed cancel reaches, against the Scope Out bullet."
  - "D8 in decision-record.md: settle what Master is told and can do when ResolveStrandGUID fails, given tender resume sends lines only to recorded sessions."
---

Focus targets the BLOCKING and MEDIUM areas of round 1; the rest of the record is reviewed as usual.
