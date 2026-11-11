```yaml
lens: fix-way-forward
outcome: reported
```

Lens: fix-way-forward. I checked that each refusal, next step and criterion the fix added can be reached from the state it names and makes progress.

**[MEDIUM] Removing the record does not get a logged-out directory off the worktree (F9)**
- **Changed text:** D2, the added sentence "A record whose directory exists but is logged out is kept too: … the way forward is to log the directory in again or remove the record so the next fresh session searches and picks."
- **Issue:** the second way forward, removing the record, leads back to the same logged-out directory.
- **Evidence:** D2's fresh-launch bullet says that with no record the launch "first searches every known directory … When any directory holds them, it records the one holding the most recently written transcript and makes no pick." The logged-out directory still exists. It still holds the transcripts of the worktree's resumable sessions, and it stays a known directory: it is a listed account's or the `env` module's. So the next fresh session's search finds those transcripts there, records that directory again and makes no pick. D2's Auto-mode assumption says the same: "removal alone moves only a worktree with none."
- **Fix:** name only "log the directory in again", or "rewrite the record with another directory". Or say that removal leads to a pick only when the worktree has no resumable session in that directory.

Every other way forward the fix introduced can be reached and makes progress:
- the `env` module as the route for a per-pane directory (F5)
- refusing a mismatched directory until the session's next launch (F7)
- reading the remaining known directories when the list fails to load (F11)
- the resume check's refusal of a gone directory, followed by `Prepare` recording what the search finds (F3)
- the counter raise (F6/F13)
- the new acceptance rows

Further tool calls: none.
