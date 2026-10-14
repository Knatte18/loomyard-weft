---
verdict: APPROVED
---

The review range is `git diff $(git merge-base main HEAD)..HEAD`: `_lyx/shed/hibernate-trigger/status.json` gives `product.parent` as `main`, there is no `_lyx/loom/rework`, and the Publish failure section reads `none`.
No finding is raised; each focus question is settled below.

- **Watch sequence.**
  `Watch` in `internal/hibernateengine/watch.go` runs ownership check, `waitForTrigger`, `advance(PhasePausing)`, `Pause`, `advance(PhaseConfirming)`, `awaitConfirmation`, a fresh `TenderStatus`, `advance(PhaseHibernating)`, `awaitWake`, `advance(PhaseResuming)` and `liftOwned`, in that order, as card 4 specifies.
  The wake is a sleep that ends after the call, seen through `pauserecord.SleepWatch` on the injected `Now` and `Awake` clocks.
  `Sleep`, `Hibernate`, `Pause`, `Lift`, `TenderStatus` and `Notify` are all `Deps` fields, and `watch_test.go` and `world_test.go` contain no real `time.Sleep` or `time.Now`.
- **Tender.**
  `Lift` matches scope, caller and `PausedAt` (via `Equal`), so it lifts exactly one entry by identity and lifts nothing for a stale or foreign ref.
  `Resume` and `Lift` share `liftEntries`, and `Resume`'s refusal text is unchanged.
  The `Added` callback runs after the record update and before the grace, never for an `AlreadyPaused` scope.
  The new `TestPause` and `TestResume` rows pin each of these.
- **Shared decision.**
  `hibernatecli/deps.go` fills `Pause`, `TenderStatus` and `Lift` from `tendercli.HubDeps` with `hibernateengine.PauseCaller`.
  The request file and `watch.log` live under `fabricengine.HubScratchDir` plus `lyxdirs.HibernateDirName`.
- **Sleep deny.**
  `sleepCommandPattern` in `settings.go` covers all five verbs with flag words, a path prefix and the shared command-position prefix, and rejects `hibernate.target`.
  `TestSleepDenyHook_DeniesMachineSleepVerbsOnly` has a denied row per verb and passing rows for `status`, `status hibernate.target` and `restart sshd`.
  `deniedRecoveryForms` and `PATTERN-no-denied-recovery.md` gain the five forms.
- **Test economy.**
  The `test-budget.yaml` entries match the top-level test counts (`internal/hibernateengine` 4, `internal/hibernatecli` 1, `claudeengine` 112), and `tenderengine` adds rows to `TestPause` and `TestResume` rather than new top-level tests.
  No duplicate or per-helper test was found, and every behavior the diff adds has a covering test.
- **Verification.**
  `lyx gate test` on `./internal/hibernateengine`, `./internal/tenderengine`, `./internal/shuttleengine/claudeengine`, `./internal/shortcutcli`, `./internal/hibernatecli`, `./internal/tendercli` and `./cmd/lyx` passed.
  `lyx gate test --tags integration` on `claudeengine`, `hibernatecli` and `hibernateengine` also passed.
- **Pattern constraints.**
  `lyxdirs` alone declares the `hibernate` directory name.
  The daemon's trigger wait is registered in `unboundedWaitFiles` and `PATTERN-daemon-wakeups.md`, and backs off from 1s to 30s.
  The daemon sends no key and no text of its own.
  The orch notice goes through `orchcli.NotifyPrime`.
  The spawn is logged through `internal/logger`.
