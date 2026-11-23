```yaml
lens: code-affected-set
outcome: reported
```

Task: code-affected-set lens. For each signature, output, key, message, file shape or behaviour the diff changes, I looked for the callers, tests, docs and rule documents it forces.

**Per change: the set found, and whether it is covered**

| Change | Set found | Covered |
|---|---|---|
| `statuscommit.Deps` loses `Push`, `Admit`, `Owed` and `Sleep` and gains `StartPush` | `battencli.commitStatusDeps` and `loomcli.commitStatusDeps`; their test fixtures in `run_test.go`, `driverdeps_test.go`, `drivewiring_test.go` and `wiring_commitstatus_test.go`; `readopt.go`, which still uses the kept `Push` | Covered |
| The batten start-row exception and the marker's `push_pending` forcing retire | `PATTERN-run-fsm.md` (its "start rows commit only" section removed); `commitstatus.go` and `commitstatus_test.go` (a marker row that still carries `push_pending`) | Covered |
| `commit-records` stops pushing; its envelope fields change (`push_skipped` and `push_stopped` give way to `pusher_started`) | The verb, its `Long` text, `docs/overview.md`, `loomcli/doc.go` and the tests. `fabriccli`'s `push_skipped` is a different verb and stays valid. | Covered |
| `loomcli` drops `commitStatusDepsGuardedBy` and `pushFailureWayForward`; the pre-push check stays on the landing path | `guardedCommit` and `guardedPush` still serve `landingdeps.go`; `prepush_test` was trimmed | The `prepush.go` comments are stale (finding F1) |
| `IsPushRejected` loses its last production caller | `cmd/lym/testonlycode_linux_test.go` allow-list entry added | Covered |
| New `PairPush.OwedUnrecorded` and `SyncOptions.OwedUnrecorded`; `ErrEntryEnded`, `TakeoversError`, `ErrTakeoverPassOwed`, `EndedHandedOver` | `IsPushOwed` and `isTakeoverStop`; `withoutFinalizeEntered`; `fabricengine/doc.go`; `pairteardown/doc.go` | Covered |
| New hub-level lock directory `lymdirs.PusherDirName` | `lymdirs/doc.go` | Covered |
| New hidden verb `lym loom pusher` | `resolvePersistentPreRun` skip; `loomcli/doc.go`; the glossary line | Covered |
| New `internal/runpusher` package | `docs/overview.md` lists it; the wake-up scan list in `cmd/lym/daemonwakeups_test.go` and `PATTERN-daemon-wakeups.md` point to it | Covered |
| `TeardownDeps` gains `Push` and `Level`, plus two sentinels | `envkit` `nilLegal`; `wire.go`; `battenshed/doc.go`; `PATTERN-pair-remote.md` | Covered |
| `ReadRun` gains `localHoldsTip` | `hubRunReader` and `runread_test`; `PATTERN-run-fsm.md` | Covered |
| `UpstreamSHA` now also wraps `ErrRefNotFound`; new `DeleteRemoteTrackingRef` | `gitrepo/doc.go` lock list updated | Covered |
| Declared PATTERN changes | The `PATTERN.md` lines and the three backgrounds all carry the declared texts verbatim. The glossary line lands. Nothing left in `pattern/` states a transition push before the run acts. | Covered |
| Tender lift wording in `tendercli/cli.go` and `tenderengine/report.go` | Updated | Covered |
| Commit trailers | `git log` over the range shows no `Pattern-Departure:`, `Loosened-Check:` or `Root-Cause:`. | See F2 |

**Findings**

- **F1 — LOW, consistency.** `internal/loomcli/prepush.go:1` and `:25–26` still describe the check as running "right before the run commits or pushes its work". `:168–171` documents `guardedCommit` as the guard for a status commit, whose "caller that pushes after its commit reads that error as the stop of the push too". After the diff, status commits and `commit-records` run no check. Only the landing's commit and push and PR-Review's `PushTask` use `guardedCommit`/`guardedPush`, and the pusher uses `guard`. Evidence: `prepush.go:1,25,168–172`; the diff's `wiring.go` change (`Commit` unguarded). Fix: reword the file header and the `guardedCommit` comment to the landing and the pusher.

- **F2 — MEDIUM, scope.** The diff removes assertions from tests that exist at the review base, and no `Loosened-Check:` trailer was found in the range (`git log` grep). The removals are:
  - the `commitStatusDepsGuardedBy` Commit/Push assertions in `TestPrePushCheckWiredIntoDeps`;
  - the `TestCommitRecordsVerb_ErrorEnvelopes` push-failure and way-forward rows;
  - `TestNew_OwedPushRetriesBehindTheGate`;
  - `TestNewCommitStatusSeam_RetriesAFailedStartChainPush`.

  Most go with retired behaviour, so a trailer may not be needed for them. But `TestCommitRecordsVerb_ErrorEnvelopes` lost the "exactly once" way-forward checks and the `FailedPushWayForward` coverage, and the `slackengine` `relay_test` change (`r.start()` to `r.settle()`) also loosens or changes a base test with no trailer. The `relay_test` change is the one the focus directive names. Evidence: `slackengine/relay_test.go:1100–1103` in the diff; `git log` trailer grep returned nothing. Fix: add a `Loosened-Check:` trailer with a measured cause on the commit that changed `relay_test`, or revert that edit.

- **F3 — LOW, consistency.** `pattern/PATTERN-run-fsm.md` ("A missing code tip waits") now says a pair that holds the tip locally is not held. The `PATTERN-pair-remote.md` background does not state that its own D6 window (records landed, code not yet) leaves origin naming a `Warp-SHA` it lacks. Neither the entry nor a declared text requires it, so I did not raise it. I looked for no other omission.

**Tool calls (4 of 6)**
1. A grep for stale push-before-act, `push_pending` and synchronous-push wording across `.md` and `.go`. The manifest lacked the untouched files.
2. A grep for `commit-records`, `push_skipped`, `push_stopped` and `guardedCommit` callers, to find stale consumers.
3. A grep for gate-admission and `statuscommit` mentions outside the diff.
4. A grep of `prepush.go` and `runfsm` for push wording, and one for `IsPushRejected` callers plus a `git log` search for trailers.
