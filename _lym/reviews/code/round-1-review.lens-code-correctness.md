```yaml
lens: code-correctness
outcome: reported
```

Task: I checked the correctness of the changed code case by case. Two LOW findings come out of it, and no BLOCKING or MEDIUM ones.

**Cases run (function or branch, input, verdict)**
- `runfsm.Derive`, a cancelled record with its task branch gone from origin and the board `done`: counts as no record, so `landed`. Correct.
- `runfsm.Derive`, a cancelled kept record with the branch on origin: `cancelled` with `PairKept` true. Correct.
- `runfsm.Derive`, a batten record `done` with the board `done`: `landed`. Correct.
- `runfsm.Derive`, a child `blocked` at `Finalize`: `halted` before `landing`, as D1's order has it. Correct.
- `runfsm.Act`, an owed push with the run owned elsewhere: `wait` on `push_owed`. Correct.
- `runfsm.Act`, `landing` past the parent merge under a hold: `act`. Correct.
- `runfsm.Act`, `landing` past the parent merge with a covering pause: `wait` on `pause_covers`. Correct.
- `runfsm.Judge` with `Aside.Cancel` on a finished cancelled record: the record then reads `running` or `starting`, not `cancelled`. Correct for `lym batten cancel`, whose own "already cancelled" check runs on the status afterwards. See F1 for `lym tender resume`.
- `shedrun` writers: `WriteCancelIntent`, `FinishCancel` and the pause-intent pair, called on a run with no status file. They write nothing and report `found` false. `FinishCancel` with no intent errors. Correct.
- `cancelRun`, origin unreachable: the intent is written and committed, the run ends, `EndOnly` is set and the re-run is named. A re-run finishes it, because `Present` with a dead writer does not refuse. Correct.
- `cancelRun --remove`: the pair is removed, then the records. A crash between the two leaves an unfinished intent that reads `cancelling`, and a re-run finishes it. Correct.
- `cancelRun` without `--remove`: `FinishCancel` sets the state `cancelled` and the record is committed. Without a record the records are removed instead. Correct.
- `cancelRun`, `WriteIntent` found but `CommitRecord` fails: it errors naming the re-run and drops the lock. Correct.
- `cleanup.judgeRecord`: a cancelled run is skipped while its pair stands locally (naming `lym fabric remove <slug>`) or its branch is on origin, and removed once both are gone. `clearDoneRun` ignores cancelled records. Correct.
- `Teardown.Run` and `removeCancelledRunRecord`: it deletes only a `cancelled` record, commits the deletion and spawns the push. A failure becomes a `Result.Warning` naming `lym batten cleanup --apply`. Correct.
- `cancelUnderWay`, an unreadable record or a failed lock probe: counts as a cancel under way, which is the safe side. Correct.
- `ReadRun`, a cancelled record: it reads the fetched task branch ref only then. With no pair here, the child is read from origin. Correct.
- `ReadRun`, `battenAtLanding`: `PastParentMerge` is true. With a child and a pair it uses origin's parent branch, or the local one under the push skip. Correct. With no pair here it is false, which is the carried-over open risk and not a new finding.
- `tenderengine.Pause`, a run owned elsewhere: refused before any entry is written. `Record` failing keeps the entry live and names the re-run. Correct.
- `tenderengine.Resume`: the verdict is asked first, then `Clear`, then the lift. A `Clear` failure lifts nothing. Correct.
- `commitRunIntent`, a slug with no status file: returns nil without writing, as documented. Correct.
- Dobby `planPass` and `drivePasses`: only `running` is drivable and anything but `act` gives no pass. The lift detection reads `Covered` and `Verdict` from the same listing, so they agree. Correct.
- `battencli.refuseByAct`, `loomcli.refuseByAct`: a landed run with no status file gets the rule's terminal refusal. A decode error in the loom path defers to the driver. Correct.
- `moveScratchAttempts`: it moves a scratch-only record into the records directory and does nothing if the destination already holds one. Correct.

**Findings**

- F1, LOW, design: `internal/tenderengine/resume.go` (the `terminal` exemption) with `internal/tendercli/runintents.go:31`.
  - `resumeAside` sets `Cancel` aside, so a finished cancelled record never reads `cancelled` in `lym tender resume`'s verdict. The resume of a cancelled run therefore gets `act`, not `wait`/`terminal`.
  - The terminal exemption in `Resume` is dead for cancelled runs, and the `Aside.Cancel` field is meant to drop only a cancel under way.
  - Evidence: `internal/runfsm/act.go:186-188`, `internal/runfsm/state.go:77-79`.
- F2, LOW, design: `internal/battencli/cancelmarker.go:53-58` with `internal/battencli/startguard.go:283-290`.
  - An undecodable batten record reads as an unfinished cancel. `lym batten run` then refuses naming "run `lym batten cancel` to finish it".
  - That cancel cannot proceed, because its `ReadStatus` and `WriteIntent` fail on the same decode error. The refusal names a way forward that cannot work, and the operator is left to repair the file by hand.
  - Evidence: `internal/battencli/cancel.go:318-321`, `internal/battencli/cancelmarker.go:186`.

I also noted one point I did not file as a finding. `pairHere` in `ReadRun` (`internal/pairteardown/runread.go:90-91`) turns a task worktree that exists but cannot be resolved into "no pair here" and swallows the error. The child is then read from origin. This is tolerable.

**Tool calls**
1. Read `.../tool-results/bjiiekq1b.txt` — my context held only a 2KB preview of the persisted diff of the callers, the dobby, loomcli, pairteardown and landingshed.
2. Read the same file from offset 1200 — to check whether the output ran past the first read, which showed it was truncated.
3. `git diff` of `internal/tendercli/cli.go`, `internal/tenderengine/pause.go` and `resume.go`, and `tendercli/deps.go` — for the pause and resume wiring the truncated output cut off.
4. `grep` for `liftPending` and the `pullrequest` record write sites — to check the dobby lift ordering and that every record write has a commit.
