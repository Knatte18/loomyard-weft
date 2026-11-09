```yaml
lens: walkthrough
outcome: reported
```

Walkthrough lens: I ran each decision's mechanism end to end on one concrete case and checked its end state against the decision's stated purpose.

**Decisions walked**
- D1 — `lym adopt pay` from a plain shell on machine B, with A as owner: caller validates, spawns worker, follows the record, prints envelope. Purpose met, but D2's record holds no envelope (finding 4).
- D2 — worker killed after `pairs`, re-run with the same subjects: resumes at `pause` using the record's moved runs. Met. A resume adding `--keep-local` restarts at `reconcile` and re-runs every step safely. Met.
- D3 — the steps in order; a lost owner write leaves the subject out of the stood list and the record ends `done`. Met.
- D4 — B's reconcile pulls A's owner record while the record is `running`; the watcher and dobby stay idle until the `owner` step's write stands. Met inside the worker. Missed on the `lym s` path (finding 2).
- D5 — a run on B holding a dobby `owner` entry: it becomes an `adopt` entry. `recheckOwnerPause` lifts only `OwnerCaller` entries, so nothing lifts it. `lym tender resume` lifts it, then `planPass` gives the full pass. Met.
- D6 — deploy fails after `owner`: record `stopped` at `deploy`, owner written, runs adopt-paused. Met for a hand re-run of `lym adopt`. Missed through `lym s` (finding 1).
- D7 — `lym s` passes `--no-orch`: the beam step starts the dobby only, then `beam start` starts the orch. Met.
- D8 — green end: no lift, no BeamUp, no pass start; it still removes the fix task and resets the try count. Met.
- D9 — `interrupted` record: the section shows the step and the re-run. Met.
- D10 — `lym s` in beam `pay`, owned by A: `--here` pulls Bolt, sees A, runs `adopt pay --no-orch`, then `beam start`, then `reed attach`. Met for a fresh adopt. Missed for an adopt interrupted after `owner` (finding 1). The exit code of a `stopped` record is unstated (finding 3).
- D11 — idle A, owner moved to B: periodic pull, probe reads B, owner pause, archive, exit. Met. The bound's description of the window is wrong (finding 5).
- D12 — `lym adopt --archives pay` lists tags whose commit the slug's origin branch lacks. Purpose at risk: which repo's origin and which branch side is unclear (finding 6).

**Findings**

1. **MEDIUM · design · D10 with D2 and D6 (`decision-record.md` lines 171–173, 137–138).** `lym s` cannot resume an adopt that stopped after its owner write.
   - Case: the worker writes B as owner, then the deploy fails or the worker is killed. `--here` pulls Bolt, reads B, ends `owned_here` and reads no record.
   - The shortcut then runs `beam start`, which brings up the dobby and orch, on the not-yet-deployed lym if the deploy is pending. The `deploy`, `beam` and `check` steps never run.
   - So "resumes after a break" fails through the entry D10 makes normal. Only `lym beam status` (D9) shows it.
   - Evidence: D10 line 172 "When no other machine owns the subject, it ends ok … reading or writing no record"; D3 puts `owner` (step 4) before `deploy` (5), `beam` (6) and `check` (7).

2. **LOW · design · D10 with D4 (`decision-record.md` lines 171, 92).** `--here`'s Bolt pull happens in the caller before any worker or record exists.
   - On the adopting machine, that pull can bring in the foreign owner while no unfinished record exists, so D4's suppression does not apply.
   - The orch watcher's `ownerTick` then stops the orch strand, and the dobby takes D11's archive-and-exit path on the machine about to adopt.
   - The support log says this was fixed: "D4 now covers D11's steps".
   - Evidence: `internal/orchengine/owner.go` `ownerTick` stops on `elsewhere`; `internal/dobbyengine/loop.go:494` exits on `beamOwnedElsewhere`; D2 line 49 says the record is written only by the worker.

3. **LOW · consistency · D1 with D3 and D10 (`decision-record.md` lines 27, 86, 177).** No decision says which exit code a `stopped` record gives.
   - D1's caller "exits with its code". D3 ends a held-back record `stopped`, and today a held-back project returns no error, so it exits 0.
   - D10's "a stopped adopt starts nothing" relies on a non-zero exit.
   - Evidence: `internal/adoptcli/adopt.go` `adopt` returns nil error for a held-back project (sets a Warning, lines 328–333).

4. **LOW · consistency · D1 and D2 (`decision-record.md` lines 27, 44–46).** D1's follower prints "the envelope the worker wrote", but D2's list of record contents has no envelope or outcome.
   - Missing: per-slug outcomes, checks, warnings, and the `--archives` pointer D12 adds to the end envelope.
   - On a resume, it is unstated whether the envelope covers earlier processes' steps.
   - Evidence: D2 line 44–46 lists the record's contents; D12 line 214.

5. **LOW · consistency · D11's bound (`decision-record.md` line 207).** The bound "a commit made after the archive, in the pause's grace, stays local" does not match the mechanism.
   - The owner pause D11 reuses waits out the 10-second grace, interrupts the sessions and ends the run's owned processes before it returns. So the archive runs after the grace, never inside it.
   - Evidence: `internal/tendercli/deps.go:213` `pauseGrace = 10 * time.Second`; `PauseOwnedElsewhere` (deps.go 227–244) pauses, then ends owned processes; it is wired as the dobby's `PauseOwned` at `internal/beamcli/dobbydeps.go:156`.

6. **LOW · design · D12 with D11 (`decision-record.md` lines 193–194, 212).** D11 archives both code and weft tips under the same `archive/&lt;slug&gt;/&lt;sha12&gt;` name. The weft is its own repository with its own origin.
   - D12 reads "the archive tags origin holds" and compares each with "the slug's origin branch", singular. It names neither repository nor side.
   - If a weft tag is compared against the code branch, it is never contained and is always listed.
   - Evidence: `internal/fabricengine/archive.go:78` `ArchiveTagName` has no side; `archiveWeftTip` pushes to "the weft repo's origin" (archive.go doc above line 109).

Tool calls:
- Read `internal/tendercli/deps.go` lines 213 and 222–245, and found the `PauseOwned:` wiring. My context did not show whether the dobby's owner pause waits out its grace or ends processes, which finding 5 depends on.
