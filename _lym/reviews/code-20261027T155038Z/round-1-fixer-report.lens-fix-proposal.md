```yaml
lens: fix-proposal
outcome: reported
```

No findings.

I tried to break each finding the fixer fixed as proposed, and none held.

- **F5 (claim budget):** `hookRead` joins the lines with `"\n"` (`internal/creelcli/hookread.go:67`), so one extra byte per line after the first is the right count. Only the first line is exempt from the newline, which matches the code. The two edge rows hold. The ids stay single-digit (1–2, then 3–4), so the prefix lengths computed from ids 1 and 2 stay valid.
- **F1 (`Prepare` hook wiring):** setting `engine.dirs.Board` after `engineWith` is safe, since the test is in the same package and the hub row's `Prepare` loads its config without error. The Windows branch converts the flag directory the same way `Prepare` does.
- **F2 (daemon-start reconcile):** the row would fail if the `ReconcileFlags` call in `Run` were removed. Beta's flag would be missing and alpha's stale flag would remain. It sets `r.geom.FlagDir` before `start`, and `options` reads `r.geom` then.
- **F3 (busy and yielded paths):**
  - The claimed-first row works because `snapshot` runs before `typeNotice`, and `typeNotice` calls `idle` first. The idle hook therefore fires between the snapshot and `reserveNotice`. With the reservation check removed, `Send` would be called and the row would fail.
  - The busy row registers `WaitMail` inside `Send`, while the notice is reserved. So it is answered by `releaseNotice`, not at once.
  - `ReadThenFail` journals through `writer.append`, so the pool state and the flag sync move together.
- **F4 (return row):** the removed receiver's message goes to the sender through `releaseGoneHolder`, so the flags end as exactly alpha's. A missing sync on that path would leave beta's flag standing, which fails the row.

Two notes, neither a defect:

- The F3 hook and `ReadThenFail` are test-only seams: `SetIdleHook` in `creelfake` and `export_test.go`. They add no production surface.
- The F4 row depends on two `sweepStep` calls for the confirm cycle. That is the same timing the existing "MailReturnsOneStepAtATime" row uses.

Tool calls:
1. `grep` for `hookRead` in `internal`, to locate the hook; my context lacked it.
2. `sed -n 56,120p internal/creelcli/hookread.go`, to confirm the hook joins lines with `"\n"` and adds nothing else to the counted bytes; my context lacked the hook's source.
