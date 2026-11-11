```yaml
lens: code-tests
outcome: reported
```

Lens code-tests: for each behaviour the diff adds or changes, the test row that would fail if it broke.

| Behaviour | Pinning test | Status |
|---|---|---|
| Stencil rules (new commit for a fix, `run_ended` end-turn, no work after `outcome: stuck`) | none | Exempt, static prose |
| Halted run holds back the Master notice, names it under `notice_held_back` with the state in the reason | `decision_test.go` `TestDecisionVerb_NotifiesLiveAgents`, row `HaltedRun_HoldsBackTheWebsterSession`; the loop asserts the reason and the strand list | Pinned |
| Running run still wakes the Master | same test, row `RunningRun_WakesTheWebsterSession` | Pinned |
| Halted run still notifies a non-Master strand | same test, `darn` in the halted row's `want` | Pinned |
| No status file (`found` false) notifies everyone | existing rows now run with `state ""`, so `found` is false | Pinned implicitly |
| `orphanedMasterRefusal` and `refuseOrphanedMaster`: a Master with free `run.lock` refuses, with `run_ended: true`, a way forward and a non-zero exit; the verb never runs | `ownership_test.go` `TestRefuseOrphanedMaster_RefusesAMasterWhoseRunEnded`, row "a Master strand whose run ended" | Pinned |
| Master under a live lock, another strand and an unnamed shell reach the verb | same test, the three `wantRan` rows | Pinned |
| Lock probe error refuses with "way forward: re-run the verb" | none | **Uncovered** (F1) |
| `addVerbs` wraps all six bracket verbs and no others | none; the test calls `refuseOrphanedMaster` on a synthetic command | **Uncovered** (F2) |
| Stuck reason counts the failing entry's re-prompts against the cap | `verifygate_test.go` `TestVerifyGateStuckReason_CountsRepromptsAgainstTheLimit`, rows "spent budget" and "failing entry's own re-prompts" | Pinned |
| Zero re-prompts with `EndedEarly`, and `EndedEarly` with re-prompts left | same test, the two `EndedEarly` rows | Pinned |
| `failedEntryReprompts` fallback to `gate.Attempts` when no entry is marked failed | none; every row marks an entry failed | **Uncovered** (F3) |
| Unreadable report: count plus way forward | same test, row "an unreadable report…" | Pinned |
| Command is named in the stuck reason; `withCommand` with an empty command | `TestVerifyGateStuckReason_NamesTheFailureAndAWayForward`, command rows and the dirty row | Pinned |
| Timeout, hang, exit-code and watchdog reasons; failing identities | same test, rows 1–5 | Pinned |
| No-test failure says so and quotes the tail; the tail is cut to the last bytes | same test, the two `LogTail` rows | Pinned (ASCII only) |
| Dirty paths: no command, own fix step | same test, dirty row | Pinned |
| Run-level `Entry` failure: findings, own fix step | same test, `drift` row | Pinned for short findings |
| `cutHead` truncation, and rune-boundary handling in `cutHead` and `cutTail` | none; no findings over 400 bytes and no multibyte input | **Uncovered** (F4) |
| Gate writes `Command` and `ExitCode`, and turns a single opaque failure into `LogTail` with no `Failures` | `TestVerifyGate_FailureNamingNoTestKeepsTheOutputTail` | Pinned |
| `command` reset per arrival (a dirty tree before render carries no stale command) | none; the dirty row writes a hand-made report | **Uncovered** (F5) |
| A failure with named tests keeps `Failures` and writes no `LogTail` | none; the new gate test covers only the opaque case | Gap, folded into F5 |

Test economy: the three new `websterengine` tests and the two others each pin a distinct behaviour, and I found no duplicates. `TestVerifyGateStuckReason_HungReportNamesTheHang` was renamed and widened in place, which is correct. The redundancy report judged none of them, so no `testtiming:keep` finding applies.

## Findings

- **F1** — MEDIUM, `design`. `internal/webstercli/ownership.go:296`, test `ownership_test.go`. The lock-probe error branch of `orphanedMasterRefusal` is untested. It refuses a Master strand on a probe failure, a deliberate choice that differs from the warn-only path. Add a row where `run.lock` cannot be probed, for example the scratch dir is a file or the lock path is a directory, and assert the "could not tell… way forward: re-run the verb" refusal.
- **F2** — MEDIUM, `design`. `internal/webstercli/cli.go:326-331`. No test shows that each of the six bracket verbs is wrapped, or that `status`, `run` and the others are not. The guard test uses a synthetic command, so dropping a wrapper leaves every test green. Add a test that builds the verb tree via `addVerbs`, runs each bracket verb as a Master strand with a free lock, and asserts `run_ended`. It should assert that a non-bracket verb such as `status` does not refuse.
- **F3** — LOW, `design`. `internal/websterengine/verifygate.go:448-456`. The `failedEntryReprompts` fallback to `gate.Attempts` is uncovered, because every test row marks an entry failed. Add a `CountsRepromptsAgainstTheLimit` row with `Entries` empty or all passed.
- **F4** — LOW, `design`. `internal/websterengine/verifygate.go:505-527`. `cutHead` is untested, including the over-limit run-level `Findings` case. The rune-boundary loops in `cutHead` and `cutTail` are untested. Add rows with findings longer than `verifyGateStuckTextBytes`, and with a multibyte rune straddling the cut, for both the findings head and the log-tail cut.
- **F5** — LOW, `design`. `internal/websterengine/verifygate.go:309-322, 414`. The per-arrival `command = ""` reset and `report.Command = command` are pinned only for the opaque single-failure path. Add a gate-level case where the dirty-tree failure arrives after a command was rendered on an earlier arrival, asserting `Command` is empty. Add a case with named failing tests asserting `Failures` is kept and `LogTail` is empty.

## Tool calls

None made. The inherited context already held the full diff and tests.
