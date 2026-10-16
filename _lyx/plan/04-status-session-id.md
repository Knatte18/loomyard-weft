# Card 4 — status-session-id

**Edit:**
- `internal/orchcli#statusFields`
- `internal/orchcli#orchCLI.statusCmd`
- `internal/orchcli#TestStatus_Envelope`

**Intent:** `lyx orch status` reports the recorded session id, empty when none (D6).

- `statusFields` adds `"session_id": st.SessionID`.
- `statusCmd`'s `Long` text names `session_id`: the orch session's recorded provider id, which `lyx orch start --continue` resumes, empty when none is recorded.
- `stop` is unchanged: it removes the strand and writes no state, so the id stays for `--continue`; `TestStart_AdoptLaunchesResumeSpec`'s continue after stop resumes the recorded id row (card 3) pins it.

Tests:

- `TestStatus_Envelope`: the "populated state" row seeds `SessionID: "sess-1"` and wants `"session_id": "sess-1"`; the "unknown tokens are null" row, with no state on disk, wants `"session_id": ""`.

**ImpactSummary:** One additive key on the status envelope, read in this repo only by orchcli's own tests and smoke helpers; stop's behavior is unchanged.
