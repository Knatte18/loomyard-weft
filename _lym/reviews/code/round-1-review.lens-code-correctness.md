```yaml
lens: code-correctness
outcome: reported
```

Task: review the diff for code correctness, one case per function and branch.

**Per-function walk (case run → verdict)**

- `ArchiveRound`, round 1 beside `round-10-*` and `harden-live-11.json` files: correct. The prefix and suffix filters keep only round 1.
- `ArchiveRound`, `os.RemoveAll` then copy: correct on rerun.
- `ArchiveFinalLedger`, no ledger: returns an error. Correct.
- `CheckCampaign`, a one-entry reviewer list: refused. Correct.
- `CheckCampaign`, a three-entry list with matching first and last entries: refused. Correct.
- `CheckCampaign`, an empty `reviewer` list: passes. Not correct (finding F6).
- `CheckCampaign`, `concurrency` 1 or `llm_driving` true: the copies key is not required. Correct.
- `NewLiveGate`, a live run that passed but started 0 binaries: red. Correct.
- `NewLiveGate`, a live run that failed with no `FAIL` line, such as a build error or a timeout: red, but `Deferred` is empty (F5).
- `NewLiveGate`, a refused copies call: recorded and the mark is untouched. Correct.
- `NewLiveGate`, a refused live call: returns an error and writes no report. Correct.
- `LiveRed`: a red report is red, a green report is green, no report plus a commits file is red, and no report plus no commits file is not red. Correct.
- `Reap`, one hub's `Owners` call fails: `continue`, so the hub's recorded processes are never passed to `End`. The `/proc` scan then ends them as unrecorded, so nothing is leaked. Correct.
- `Reap`, `StopServer` fails: the hub list keeps the entry, but `emptyRoot` still removes the hub directory. The next reap stops the server by key, so it recovers. Correct.
- `Reap`, caller and ancestors: protected through `ListStats`. Correct.
- `namesRoot`, a path `root2` and a path `root/x`: separator boundary respected. Correct.
- `WrapRow`, review row resumed after its review was written: keeps round N (`FixedRound`+1). Correct.
- `WrapRow`, fix row resumed after the process died mid-row: `Head` is re-read, so `RoundCommits.Head` is wrong (F2).
- `WrapRow`, inner error: still reaps at the end, no commits file, no archive. Correct.
- `Campaign.Halt`, no review file yet: `ReviewRound` is 0 and `harden-strays-0.json` is written. Harmless.
- `Rotate`: `rounds` below 1 or an empty list gives `[]`. Correct.
- `Rotate`, `cycled(list, round)`: `Rotate(list, round)[round-1]`. Correct.
- `copyLease`, tier slots 0: `(0, 1)`, one copy over every slot. Correct.
- `copyLease`, tier slots above the pool: `ran` is 1 and the lease clamps. Correct.
- `mergeCopies`: first non-zero exit code, `Launches` summed, `Outcome` derived. Correct.
- `runWithSlots` with `--copies` inside an inherited slot: refused before any acquire. Correct.
- `runWithSlots` with `--copies` on a vet or guard step run: copied too (F3).
- `hardenLiveRun`, empty stdout: the last-line JSON parse fails and the run is not refused. Correct.
- `hardenLiveEnv`: drops `InheritEnv`. Correct.
- `withHardenBudget`, `hardener.yaml` fails to load: falls back to the template with a warning. Correct.
- `treeScopeSettings`, hardener run: `VerifyBudget{}` has `Limit` 0, which turns the budget off (`verifybudget.go:25`). Correct.
- `ringReaches` and `segmentBouncerReaching`: correct. A missing Bouncer, a missing edge and a Bouncer lying before the record row are each refused.
- `DropRetiredRoles`: empty file unchanged, bare and `role/model` keys removed, a second pass is a no-op. Correct.
- `StopServer`, probe exits non-zero: stale socket removed, returns false. Correct.
- `clearWorktreeSandbox`, no sandbox root: starts nothing. Correct.
- `clearWorktreeSandbox`, clear fails: returns an error and the worktree is kept. Correct.
- `liveRedDetail`, `LiveRed` errors: escalates with cause budget rather than accepting. Correct.
- `escalateWith`, `OnHalt` fails: warns and adds the way forward to the detail. Correct.

**Findings**

**F1 — MEDIUM, design.** `internal/hardenshed/archive.go:40` and `internal/loomcli/hardenwiring.go:424`: the round archive is written under `WorktreeRoot/_lym/harden`, which is `location.WorktreePath()`, but committed with `CommitAnchoredPaths`. That function takes anchor-relative paths (`internal/fabricengine/commitweftpaths.go:96`).
- On a hub whose anchor is not `.`, the files land outside the anchor's `_lym`, and the pathspec `_lym/harden/round-N` names a path that does not exist under the anchor. The commit then fails, or commits nothing.
- The same applies to `ArchiveFinalLedger`.
- Every test uses anchor `.`.
- Fix: build the archive directory from the anchor path, or pass `AnchorPath` as `ArchiveDeps.WorktreeRoot`. Keep `WorktreeRoot` for the package checks only.

**F2 — LOW, design.** `internal/hardenshed/round.go:645`: a fix row resumed after a crash re-reads `Head` at resume.
- Commits the earlier call made are then outside `RoundCommits.Commits`, so the facts' "Fix commits" undercounts (D6).
- Fix: persist the row's starting head in a round file at first entry, and read it back on resume.

**F3 — LOW, design.** `internal/gatecli/run.go:659` (`copying := e.request.copies > 0 && !empty`): the copies expansion also applies to vet-step and guard-step runs, which pass through `runWithSlots` (`run.go:625`, `run.go:626` `vetStep`/`guardStep`).
- A vet or guard step would run N times and lease N times its slots.
- This is the one finding I could not fully verify: it depends on the hub's vet or guard step being planned for the key.
- Fix: set `copying = false` when `run.vetStep || run.guardStep`.

**F5 — LOW, design.** `internal/hardenshed/live.go:646-648`: a red mark that comes from a failed run with no `FAIL`/`panic:` line, from a no-binary run, or from a timeout leaves `Deferred` empty.
- The facts then print "Deferred by the red live report: none" beside a red mark, so the judge and the carry-over see no deferred finding for a spent-attempts red gate (D4).
- Fix: when the mark is red and `FailingLines` is empty, add one synthetic deferred line, such as "`harden-live` failed or started no test binary; see `<report path>`".

**F6 — LOW, design.** `internal/hardenshed/campaign.go:286` (`reviewerRepeat`) and `internal/hardenshed/config.go` (`checkValues`): an empty `reviewer` or `fixer` list, settable from a task's `run_settings`, passes Preflight and loading.
- `Rotate` returns an empty list, and `pickRound` (`internal/burlerengine/models.go:26`) returns the zero `ModelChoice`, so rounds run on an unspecified model.
- `RoundModels` is then recorded as empty strings, and D6's "reviewer model differs" rule cannot hold.
- Fix: refuse an empty `reviewer` or `fixer` list in `CheckCampaign` and `checkValues`, with the `run_settings` write as the way forward.

Findings are numbered F1–F6; there is no F4 because one candidate was dropped on verification.

**Tool calls**
1. `git grep` of `CommitAnchoredPaths`: my context did not show whether its paths are anchor-relative.
2. `git grep` of `runWithSlots`, `vetStep` and `guardStep` in `gatecli/run.go`: to see whether vet and guard steps pass through `runWithSlots`.
3. `git grep` of `VerifyBudget` `Limit` in `loomshed`: to confirm that a zero `Limit` turns the budget off.
4. `git grep` of `RoundModels.Pick` and `pickRound` in `burlerengine/models.go`: to see what an empty model list returns.
