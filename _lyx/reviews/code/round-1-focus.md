---
round: 1
focus:
  - "internal/gitexec/gitexec.go childEnv: does every spawn path (local, remote, runCoreWithin and any other caller of childEnv) get the entry, and can a caller's extra slice carry GIT_OPTIONAL_LOCKS and override the forced 0?"
  - "Does any production caller rely on git writing back the refreshed index through an optional lock, for example a status or diff probe whose result depends on a refreshed stat cache, so the change alters behavior beyond the stated bound?"
  - "internal/gitexec/gitexec_test.go new row: does the `alias.lyxenv=!env` check work where `env` may be missing or the shell differs (Windows), and does it avoid t.Parallel correctly given t.Setenv?"
  - "doc.go and the childEnv comments: do they match the decision record's decision and bound, and follow PATTERN-comment-line-breaks?"
  - "Scope: the diff touches only internal/gitexec, and the smoke test and weft write lock are left alone as the record's Scope requires."
---

Seed focus from the board entry and decision record.
