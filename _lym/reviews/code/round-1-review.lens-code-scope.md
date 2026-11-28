```yaml
lens: code-scope
outcome: reported
```

Task: trace each of the diff's changes to the board entry's goal points, and each goal point to the changes that build it (code-scope lens).

**Decisions** (the board entry's `## Goal` → `## Change` points; the record has no `Added after Discussion` entries)
- G1: move the `MailAgent` rows to a tier1 loomcli test over a real creel daemon.
- G2: teach `tmuxrec` to record and match a call's stdin, and to match a paste's buffer name by pattern, with a paste recording and a replay test over it.
- G3: move `envkit`'s session stand-ins onto the recorded tmux kit, using paste replay where they send text.

**Changes → decision**
- C1 `internal/landingshed/prreview_wait_test.go`: drops the three mail rows, `waitWorld.mails`, the mail seam and the no-mail `check` closures. Traces to G1.
- C2 `internal/landingshed/prreview_test.go:717`: the shared deps' `MailAgent` now fails the test when called. Forced by C1, so the no-mail rows still check that no mail goes out.
- C3 `internal/loomcli/prreview_mail_integration_test.go` (new). Traces to G1.
- C4 `internal/reedengine/io.go`: exports `InjectBufferPrefix`. Forced by G2: the replay pattern needs the prefix as one named constant.
- C5 `internal/testkit/tmuxrec/tmuxrec.go` and `record.go`: `Call.Stdin`, recording and binding stdin, matching on it, `holdInjectBuffers`, `refuseLocked`, buffer placeholding in `placeholdCall`, and the package doc. Traces to G2.
- C6 `testdata/session-paste.json`, the `session-paste` scenario, `TestReplay_MatchesAPasteByItsTextWhateverBufferItNames` and `TestPlaceholdCall_HoldsAPastesBufferNameAndKeepsTheNumbersOfItsText`. Traces to G2.
- C7 `tmuxrec.SessionSpec` exported and moved out of `scenarios_test.go`. Forced by C3, which seeds a session under the recorded spec.
- C8 `tmuxrec.IdleSession` and the `replaySession` split. Traces to G3.
- C9 `session.go` `newSession` shell fallback. Forced by G3: `IdleSession` never skips, and `FullEnv` and `LandingDeps` callers on a host without the shell would otherwise fail. It also changes `PlaySession` and `RecordSession` on such hosts, but in no way the move does not need.
- C10 `internal/testkit/envkit/envkit.go`: `replayedSessions`, `LandingDeps(t, dir)`, and the PR-Review and sampler `Sessions`/`SessionLive`/`RegisterSession` seams. Traces to G3.
- C11 the `LandingDeps(t, …)` callers in `rownotice_test.go`, `wiring_test.go` and `loomrecipe/fixture_test.go`. Forced by C10's signature change.
- C12 commit `c391c9f35`, comment line breaks. Forced by the comment lint on C1–C10.

**Decisions → changes**
- G1: C1, C2, C3 (C7 supports it). Built.
- G2: C4, C5, C6. Built.
- G3: C8, C9, C10, C11. Built only in part; see finding 1. "Paste replay where they send text" holds trivially: no session seam that `envkit` fills sends text, and nothing drives those sessions.

**Findings**

1. MEDIUM, class `scope`, `internal/testkit/envkit/envkit.go:474`.
   G3 is built only in part: some of envkit's session and strand seams are still hand-written closures.
   - `ImplementerSeam.LiveStrands` returns `nil, nil`. It reads reed's live strand names, the same table the moved `SessionLive` seams now read through `reedengine.SessionLiveness`.
   - The batten child deps' `ReviveStrands` (`envkit.go:661`) returns nil, and `DriverStrand` (`envkit.go:657`) returns `ChildDriverNone`.

   The board entry asks to move "envkit's session stand-ins" with no narrowing to PR-Review and the sampler.
   `_lym/landing/summary.md` narrows it to "envkit's PR-Review and sampler session seams" without saying why the rest stay.
   Evidence: `envkit.go:474` and `envkit.go:657-661`, read this turn; the decision record's `## Goal` `## Change` third bullet.
   Fix: move `LiveStrands` onto the replayed reed's strand table as `SessionLiveness` does, and the batten strand seams too if they read reed. Otherwise state in the change description why each one is not a session stand-in.

No other change lacks a trace, and no other decision is left unbuilt. The `MailAgent` closure in `internal/loomshed/samplersession_test.go:164` belongs to the sampler session tests, not the PR-Review ones, so it is outside G1.

Tool calls:
- grep of `MailAgent` across the test files and of the session and strand seams in `envkit.go`: the manifest held neither the remaining `MailAgent` closures nor the full set of envkit's session seams.
- `envkit.go` lines 462–476 and 650–668, plus `StopConflictSession` in `landingshed/deps.go`: to name the leftover strand seams and their values, which the manifest's envkit read did not cover.
