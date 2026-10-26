```yaml
lens: code-affected-set
outcome: reported
```

**Task:** code-affected-set lens. For each behaviour the diff changes, I looked for what else reads or asserts it.

**Per change, the set found**

1. **`TmuxCmd` EAGAIN retry (`run`, `runStdin`, `output`, new `pause` field)**
   - Covered. Every tmux call goes through these three methods, so there are no other exec sites to update.
   - `run` now delegates to `runStdin(nil, …)`, and a nil stdin leaves `cmd.Stdin` unset, as before.
   - `internal/reedengine/doc.go` is updated.
2. **`start_retries` key**
   - Covered where the repo enumerates keys: `template.yaml`, `Config`, `ConfigSpec`, `LoadConfig`, `config_test.go:371-373` and `shuttleengine/doc.go:490`.
   - Not covered: tests that run a start against a provider that dies, with the default template, now get the retry (see finding 1).
3. **`abandonStartup` error text and strand-removal retry**
   - Covered. The asserters of the changed strings (`startup_test.go:396/423/519-524`, `run_test.go:2095`) sit in the diff's test files.
   - `send.go:205` and `send.go:691` build their own "The pane's last lines:" text and share only `captureTail`, so they are unaffected.
   - `tenderengine/resume_test.go:76` fabricates its own error and is unaffected.
4. **`launch` and `start` split (`run.go:545`)**
   - Covered. `launch` has one caller, `start`.
   - Callers match the `ErrNotStarted` wrap (`shedadapters/burlerfix.go:245`, `websterengine`, `loomcli/driverlaunch.go:30`); the not-ready error text starts the same as before, so they are unaffected.
5. **`heldTurnEnd.background` and the notice withheld (`wait.go:891`, `hold.go:108`)**
   - Covered. `run.waitingTurnEnd` is read in both held paths: the gated unaware-shells branch and `expiredTurnEnd`. Both reuse the struct built at `wait.go:891`.
   - The plain held turn end built at `wait.go:912` leaves `background` false, so it is notified as before.
   - `RequirementBackgroundWake` is read through `meetsAll` / `RequirementSupporter`, so no concrete engine is named.

**Findings**

1. **MEDIUM, consistency.** Location: `internal/loomcli/smoke_helpers_test.go:99-102` with `internal/shuttleengine/template.yaml:6`.
   - `providerlessShuttleConfig` rewrites only `startup_timeout_s` and keeps the template's `start_retries: 1`.
   - Its providerless fixtures end a start `died` with the strand removed, which is now retried.
   - Each failing start therefore costs a second startup window plus the 5 s `startRetryDelay`, which the header there says is meant to keep the smoke suite bounded.
   - Whether the real-clock smoke tests now run slower or time out is unverified.
   - The fix is to pin `start_retries: 0` there.
2. **LOW, consistency.** Location: `docs/overview.md:408`.
   - The shuttle row says a provider that never comes up is torn down and reported as an error from `Start`, or as `died`/`timeout` from `Run`.
   - It does not mention the died-start relaunch or the `start_retries` key.
   - Whether this row has to carry per-key detail is a judgment call.
   - Under CLAUDE.md's docs-land-with-the-code rule, `doc.go` is the module doc and is updated, so this is only a possible gap.

**Not findings**
- No stencil changed.
- No other file lists shuttle config keys; the `startup_timeout_s` sites elsewhere are comments or the smoke override.
- `ErrNotStarted` wrapping is unchanged.

**Tool calls (3 of 6)**
1. Grepped the repo for `startup_timeout_s`, `start_retries`, the changed error strings and the `launch` / `abandonStartup` / `RemoveStrand` callers. My context lacked sites outside the diff.
2. Grepped the key sites, the `RequirementBackgroundWake` users and the `ErrNotStarted` consumers. I needed to see how other packages configure or depend on them.
3. Read `smoke_helpers_test.go` around the override and `docs/overview.md`'s shuttle row. I needed to know whether the smoke fixtures pin the retry and whether the docs mention it.
