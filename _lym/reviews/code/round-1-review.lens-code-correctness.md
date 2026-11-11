```yaml
lens: code-correctness
outcome: reported
```

**Task:** correctness lens over the adopt worker, steps, caller, follow, pause and record code. Per-item list first, then the findings.

**Per-item list (case run, verdict)**
- `runWorker`, lost lock: `TryLock` returns false, so it returns before any `WriteRecord`. Correct. A direct call with a different subject set is refused by `judgeStanding` before any step. Correct.
- `runWorker`, fresh record: the old record is `done` or absent, so it starts at `reconcile` with the resolved subjects. Correct.
- `runWorker`, resume with `--keep-local` added: `mergeRequest` sets `restart=true` and `from` stays `reconcile`. Correct. `--drop-local` added to a record holding `--keep-local` gives drop alone. Correct.
- `runWorker`, resume where `ResumeStep()` is `owner` or earlier: `from = reconcile`, but `record.Steps` and `StoppedPart` are cleared. This is wrong, see F1.
- `runWorker`, resume past `owner` (stopped at `deploy`): `from = deploy`, and the inputs and steps are kept. Correct.
- `runWorker`, resume of `--all` or a named request: `namedEntries` is re-run on the live board. This is slightly wrong, see F3.
- `runSteps`, step failure: the part is stored through `stop(name, err, raw)`. `recordStep` clears it on later success, and `envelopeOf` merges `StoppedPart` after the done steps without listing the step. Correct, and it matches the Added-after-Discussion entry. An empty part (`{}`) stores nil. Correct.
- `runSteps`, `refuseBeforeSteps` refusal: `stop(from, err, nil)` runs no step and keeps the earlier steps. Correct.
- `envelopeOf` with `InFlight == ""` and no `StoppedPart`: the merge is skipped. Correct.
- `stepPause`: with no adoptable subject and no root going it returns nil. A pause-record update failure returns a stopError, before `Inputs.Paused` is touched. Correct.
- `pauseEntries`:
  - No entry: a new `AdoptCaller` entry with `GraceEnd = now`. Correct.
  - `OwnerCaller` entry: converted in place, keeping the pause time. Correct.
  - Entry of another caller, `adopt-hold` included: kept and reported in `other`. Correct.
  - Slug already in `listed`: skipped. Correct.
  - Beam with no owner entry: no entry. Correct.
  - Beam with an owner entry: converted and listed as paused. Correct.
- `pauseTargets`: the unstarted non-front released task and the run filters (stopped, error, warning) are right. A parent that is an adopted beam or the prime is reached through `writeReaches`. Correct.
- `stepOwner`: a `SetOwners` error returns `ownerWriteError`, with `OwnerStood` already reset to nil. That is consistent, since nothing stood. Correct.
- `stepDeploy`: a failure stores the deploy outcome as the stopped part, and the owner and pauses stand. Correct.
- `stepBeam` and `stepCheck`: both read only `Inputs`. Correct.
- `stepDone` with a held-back project: stops with the re-run command for the held subjects only. This is wrong, see F2.
- `runAdopt` / `follow`:
  - Verdict follow keeps `exited == nil`, so a vanished record errors at once. Correct.
  - A spawned worker that lost the lock to a later same-subject worker is followed (`Start.After(since)`). Correct.
  - A spawned worker that lost the lock to another-subject worker is refused naming those subjects. Correct.
  - A worker that exited with only a stale record standing: `missed` gives one more wake, then the re-run error. Correct.
  - The `interrupted` record returns an error naming the re-run and `--discard`. Correct. The `stopped` record gives a non-zero `ErrFields`, `done` gives ok. Correct.
  - The cap is checked after the done and stopped reads, so a finished adopt is never reported as still running. The backoff resets on a fingerprint change. Correct. The final sleep can overshoot the cap by up to 8s, which is harmless.
- `discard`:
  - Lock held: refused, record untouched. Correct.
  - Undecodable record: removed. Correct.
  - Done or absent record: nothing removed. Correct.
- `WorkerAlive`: empty start ticks falls back to the pid alone. Correct.
- `adoptSectionOf`: the `done` state returns nil, and each of the three states gets its way forward. Correct.

**Findings**

- **F1, MEDIUM, design.** `internal/adoptcli/worker.go:226-228`: a resume whose first step not done is `owner` or earlier starts at `reconcile` and sets `record.Steps, record.StoppedPart = nil, nil`.
  - D2 says a resuming worker "rewrites only the pid, start ticks, start and state ... keeping the done steps and stored inputs", and `recordStep` already replaces a step of the same name.
  - Clearing the steps drops the earlier process's step parts from the end envelope. The lost parts are:
    - the `pairs` step's mutations, so `mutations` and `partial` under-report;
    - the earlier `pause` step's `AdoptPaused`, `Run = pausedRun` and `PauseStanding` slug fields. The re-run `pause` step skips the slugs in `Inputs.Paused`, so nothing re-sets those fields.
  - Fix: clear `Steps` only on a fresh record (`verdictFresh`), or only when a `--keep-local`/`--drop-local` restart changes how the reconcile answers. Otherwise keep the done steps and let `recordStep` replace each as it re-runs. Add a test row for a resume that restarts at `reconcile` and keeps the earlier pause part in the envelope.

- **F2, MEDIUM, design.** `internal/adoptcli/steps.go:318-328`: the stop at `done` names, as its way forward, a re-run of the held-back subjects only (`rerun.Slugs = heldBack`, `All = false`).
  - The stopped record keeps the whole adopt's subjects, or `--all`.
  - `judgeStanding` (`worker.go:136`) and `Request.SameSubjects` (`adoptrecord/record.go:70-75`) therefore refuse the named command as "other subjects". It is refused unless the held-back set equals the record's subjects.
  - The same unusable command is shown by `lym beam status` (`beamcli/adoptsection.go:49-53`).
  - The usable path is `&lt;the record's own command&gt; --keep-local`, which restarts at `reconcile`. The alternative is `lym adopt --discard` first.
  - Fix: build `keep` and `drop` from `run.Req` (same slugs, root and all) with only the flag changed. Alternatively let `SameSubjects` accept a call whose subjects are a subset of an unfinished record held-back set. Add a test that follows the stop's own way forward through `judgeStanding`.

- **F3, LOW, design.** `internal/adoptcli/worker.go:214`: on a resume, `namedEntries` re-resolves `--all` against the current board instead of using the recorded `Subjects`.
  - D2 records the resolved subject set. A beam created or landed between the interruption and the resume changes `run.Slugs`.
  - `run.Slugs` then differs from `record.Subjects`, and a done beam now fails the named-slug check.
  - `stepCheck` and `refuseBeforeSteps` use `run.Slugs`, while the envelope uses `record.Subjects`.
  - Fix: on `verdictResume`, take `slugs` and `rootNamed` from `record.Subjects` (`root` splits out into `rootNamed`) rather than re-resolving.

**Tool calls**
1. `grep -n -A14 "^type OriginReport struct" internal/fabricengine/*.go` and `grep -n -A60 "^func namedEntries" internal/adoptcli/*.go`: one Bash call. I needed `OriginReport`'s fields to check `reconcileWire` and the `namedEntries` resolution on resume, and neither was in my context.
