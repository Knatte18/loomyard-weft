---
round: 5
focus:
  - "Hook append in D1 of `_lyx/discussion/decision-record.md`: the Bound claims the rewrite adds no new way to corrupt a Stop or ask line. Settle whether that holds when a bash `printf '%s\\n'` writes a large payload in 4096-byte chunks where today's `cat >>` writes it in one write, and when UserPromptSubmit, StopFailure, Notification and SessionEnd also append to the same file. Check the Bound and the acceptance criteria state the same behavior."
  - "The `dead` state in D3: a SessionEnd with reason `clear` or `resume` fires inside a process that keeps running, and the orch clear cycle types `/clear`. Settle what state the reducer gives such a session end, and whether precedence rule 1 and the reducer acceptance rows name that case."
  - "The orch watcher's inputs in D3, D4 and D5: settle where the orch watcher gets the session id that the registry lookup needs, given the orch has no run record and its session id changes at each clear, and whether the acceptance criteria state the choice."
  - "The claim in D3 that signal-only history entries are the same for every reader: settle whether `interrupt` and a transcript-marked `api-error` rest only on signals, given that each reads the transcript at reading time and the transcript reader reports only the newest entry."
  - "D3 against D6 on what bounds a call-time state: settle whether the phrases naming the bounding history entry in the two Decisions mean the same entry, given that the entry before a call-time state is usually a turn end the signal-only definition excludes."
---

The focus lists where to look; the reviewer still judges the whole pair against the rubric.
