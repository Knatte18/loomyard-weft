```yaml
lens: fix-claims
outcome: reported
```

Fix-claims lens: I checked whether every claim added or rewritten by the fixes holds against the code at HEAD.

**LOW**: card 15, the rewritten store-failure row ("advances the fake clock past the next UTC midnight and removes the journal directory").
- **Summary:** removing the journal directory while the current day file is still open may not be possible on Windows, which the repo supports.
- **Evidence:**
  - `journalWriter.close` runs only inside `rotate`, so at the moment the interrupt function runs, `w.file` still holds today's file open (`internal/creelengine/journal.go:237-244`, `262-264`).
  - On Windows, Go opens files without `FILE_SHARE_DELETE` as far as I know, so `os.RemoveAll` on that directory fails. This is not verified.
  - Windows support is evident from `update-plugins.cmd`, the `internal/fslink` junction rule, and `verbs_test.go` being plain `//go:build tier0` with no OS restriction.
  - On Linux the claim holds: `rotate` closes the old file, `os.OpenFile` with `O_CREATE` in a missing directory fails, and `append` returns `creel journal: open …`. That error is wrapped, with no nil dereference.
- **Possible change:** an interrupt function that advances the clock and creates a directory at the next day file's path. `os.OpenFile(..., O_WRONLY)` on a directory fails on both OSes.

These claims hold:
- **F3:** `interruptTarget` has exactly two callers, `Inject` (`verbs.go:426`) and `Interrupt` (`verbs.go:529`). So "`Interrupt` and `inject --interrupt` answer as they do today" holds if the typed error keeps today's message and its `%w` unwrap. The `default` case returns only errors from `delivery.Interrupt`, which matches "a key the delivery failed to send".
- **F2:** the `UserPromptSubmit` recording hook appends the full payload (`cat >> events`, `settings.go:286-289`, `315`), so a typed bell's prompt is in `events.jsonl`.
- **F4:** `tooShort` is an existing field of the SessionIdle test's rows (`session_test.go:259`).
- **F5:** production writes `Provider` only as `engineName` from shuttle (`open.go:88`, `run.go:600`), and only `claude` is registered, so "only in tests" holds.

Tool calls:
1. `recordingHook` grep, `verbs_test.go` header and `update-plugins.*` listing: to see how UserPromptSubmit payloads are recorded and whether Windows applies. My context had neither.
2. `recordingHook` again, plus the SessionIdle test's row fields: the first grep missed the closure, and I hadn't seen whether `tooShort` exists in that test.
3. `recordingHook` closure in `settings.go`, plus a prompt-field grep in `signals.go`: needed the closure body to see what each hook records.
4. `recordingCommand` body: to confirm the full payload is appended to the events file.
5. `shuttleengines` listing and `Provider:` writers: to check F5's "only in tests" claim against production writers.
