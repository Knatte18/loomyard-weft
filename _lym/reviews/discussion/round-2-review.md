---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D1 (process file and join rule) and D3 Bound (Windows)
    summary: D1's join test needs the runner's recorded start time, which proc.StartTime never answers on Windows, so a Windows front can never join a live runner and a gate longer than the cap never returns a verdict
    fix: In D1, say the process file holds the runner's pid and a start reading that tells a reused pid apart on every platform, proc.StartTime on Linux and proc.CreationTime on Windows, and that the join compares it with the same reader. Add a D1 acceptance criterion that a Windows front joins a live runner and supersedes one whose pid was reused. Extend D3's Windows bound to cover the join as well as the end.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D1 (claim wait, AwaitAny) and D4 (cap validation)
    summary: A front blocks on the key lock and the up-to-30 s claim wait before its cap-long AwaitAny, while D4 only checks the cap is below the lifetime, so a valid cap such as 290s on a 5-minute lifetime returns the agent to a cold cache
    fix: In D1, state that the cap counts from the front's start, so the key-lock wait and the claim wait are spent inside it and AwaitAny waits only what remains, with the lease deadline taken from that same start. Add a D1 acceptance criterion that a starting front whose runner claims late still answers within its cap.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D3 (runner killed without an exit file) and D6 (line kinds)
    summary: A card-verify runner killed by SIGKILL, an OOM kill or a crash leaves its line's process group running, since verifyrun.Run sets no parent-death signal, so a --wait gate child keeps its own slots and tiers outside any lease, against D3's claim that the kill frees the slots
    fix: In D6, state that a card-verify runner starts each line so that the line's process group is killed when the runner dies (a parent-death signal on the line's shell on Linux, as proc.StartTied gives tiers), and state the bound that remains where no such tie exists, such as Windows. Narrow D3's sentence on a runner killed without an exit file to the paths where that holds.
  - id: F4
    severity: LOW
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D5 (Abandoned set by execution.report)
    summary: The build step's record is written in build.go and a blocked tier's record in reportBlocked, outside the record path D5 marks, so a call cancelled during its last tier's build counts as a full call
    fix: In D5, set Abandoned in execution.writeRecord, which every record of a call goes through, the build step's and a blocked tier's included, rather than in execution.report; extend the D5 acceptance criterion to a call cancelled during a build step.
  - id: F5
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D1 (start failure, late claim refused)
    summary: The runner checks the lease and then creates its process file with nothing ordering the two against the front's expiry, so a claim made just after a 30 s start failure succeeds instead of being refused
    fix: In D1, state that the runner re-reads the lease after it creates its process file and ends, writing a marked exit file, when the lease has passed, so a claim after the front's expiry ends before its slot wait.
  - id: F6
    severity: LOW
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D5 (Abandoned) and Constraints (PATTERN-tolerant-records line)
    summary: lym gate report totals every record as a real outcome, so tiers the new lease, supersede and signal cancels cut are counted as failures and wall time, and the record never says how report treats Abandoned
    fix: In D5, state that lym gate report leaves records carrying Abandoned out of its pass, fail and wall-time totals, and add that to the D5 acceptance criterion beside the line that records without the field still total.
  - id: F7
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D6 (key) against D1 (join key)
    summary: D1 keys a run by the argument words as given while D6 keys card-verify by the card id, and runCardVerify accepts both 4 and 4-slug for one card, so it is unclear whether the two forms join one run
    fix: In D6, state that the card-verify key is the verb, the resolved run directory and the matched card's own id (card.ID()), so every accepted form of one card joins one run.
  - id: F8
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D1 (runner verb paragraph) against D6
    summary: D1 says the runner runs the request with --wait semantics, slot wait and tiers included, which describes only lym gate test; card-verify has no --wait flag and no tiers, and D6 runs its line through cardverify.Runner
    fix: In D1's runner-verb paragraph, say that for lym gate test the runner runs the request in-process with --wait semantics, and that for lym gate card-verify it runs the card's line as D6 states.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D3 (lease monotonicity sentence) against D1 (start failure)
    summary: D3 calls the supersede expiry the one write that moves the lease earlier, but D1's start failure also expires it
    fix: In D3, say the supersede expiry and the start-failure expiry (D1) are the writes that move the lease earlier.
  - id: F10
    severity: NIT
    class: consistency
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D5 (wall time)
    summary: The wall time spans the records from the first WaitStart, leaving out the config load, scope computation and guard scan the front also blocks through, so the claim that it covers what the front blocks through overstates it
    fix: In D5, say the wall time covers the slot waits and tiers and leaves out the pre-tier phase, and drop the clause that it covers what the front blocks through.
---

### [BLOCKING] Windows fronts can never join a live runner

**Location:** `_lym/discussion/decision-record.md` D1, the process file and join rule (lines 48 and 51), and D3's Windows bound (line 106).
**Issue:** D1 joins a run without an exit file only when the process file "names a live process with the recorded start time". On Windows, `proc.StartTime` returns `"", false` (`internal/proc/proc_windows.go:54-55`), so the runner records no start time and a front can never confirm the runner is live.
Every re-run then supersedes the live runner and starts afresh, so a Windows agent whose gate outlasts the cap re-runs forever and never gets a verdict.
That defeats the Goal on a platform the record itself supports (D3 states a Windows bound).
If an empty start time were instead taken as a match, a reused pid would pass as live.
`proc.CreationTime` (`proc_windows.go:60`) exists to tell a live process from a reused pid on Windows, so the fix is a choice the writer can make.

### [MEDIUM] The claim and lock waits fall outside the cap D4 validates

**Location:** `_lym/discussion/decision-record.md` D1 (lines 49 and 57) and D4 (line 111).
**Issue:** A starting front holds the key lock until its runner claims, for up to 30 s, and only then waits on the exit file "for at most its cap".
A joining front can first wait for that lock while another front holds it.
D4 accepts any cap below the role's lifetime, so `keep_warm_cap_roles: {implementer: 290s}` on the 5-minute lifetime passes validation, yet a starting call blocks about 320 s and the agent comes back to a cold cache.
The declared `PATTERN-keep-warm-cap` makes the resolved cap the bound of the agent's wait.
Three lenses and the handler found this independently.

### [MEDIUM] A killed card-verify runner leaves its line running

**Location:** `_lym/discussion/decision-record.md` D3 (line 95) and D6 (lines 148-152).
**Issue:** D3 says a runner killed without an exit file "frees its slots with its OS file locks".
That holds for a `lym gate test` runner, whose tiers start through `proc.StartTied` with `Pdeathsig=SIGKILL` (`internal/proc/tied_linux.go:18-19`).
A D6 runner runs its line through `verifyrun.Run`, which puts the shell in its own process group with no parent-death signal (`internal/verifyrun/run.go:76-83`).
A SIGKILL or OOM kill of the runner therefore leaves a `--wait` gate child running to its end, holding its own slots outside any lease, and a non-gate line running on without the slot it took.
The next front supersedes the dead runner and starts a line that queues behind the orphan or runs beside it.
`runproc.End` reaches descendants, so only the unsignalled kill path is affected.

### [LOW] Abandoned misses build and blocked-tier records

**Location:** `_lym/discussion/decision-record.md` D5 (line 132).
**Issue:** D5 has `execution.report` set `Abandoned`. The build step's record is written directly through `writeRecord` (`internal/gatecli/build.go:63-70`), and a blocked tier's record is built in `reportBlocked` (`build.go:83-90`), outside `report`'s own record path (`internal/gatecli/run.go:483-485`).
If the cancel kills the build in front of the last listed tier, `reportBlocked` writes that tier's record unmarked, and the call counts as a full call.

### [LOW] A late claim is not refused

**Location:** `_lym/discussion/decision-record.md` D1 (lines 50 and 55).
**Issue:** The runner reads a valid lease at 29.9 s, the front expires it at 30 s and exits 1, and the runner's exclusive create of its process file succeeds at 30.1 s.
D1 says such a late claim is refused, but nothing orders the claim against the expiry. The run lives until its next lease read and may enter its slot wait first.

### [LOW] lym gate report counts abandoned tiers as outcomes

**Location:** `_lym/discussion/decision-record.md` D5 (line 132) and the PATTERN-tolerant-records Constraint (line 187).
**Issue:** D3 adds frequent cancel sources: a passed lease, and a supersede on every tree change between calls.
`execution.report` writes the cut tier with its exit code (`internal/gatecli/run.go:479-496`), and `report.go:147-148` totals every record.
D5 adds the mark, but the record never says the report reads it, so the per-key failure and wall-time totals absorb cut tiers.
The verdict is unaffected, so this is LOW.

### [LOW] Card-verify's join key is ambiguous

**Location:** `_lym/discussion/decision-record.md` D6 (line 145) against D1 (line 45).
**Issue:** D1 keys a run by "the argument words as given", D6 "by the verb and the card id". `runCardVerify` matches a card by either `4` or `4-<slug>`, so the record leaves open whether the two forms share one run.

### [LOW] D1's runner description fits only lym gate test

**Location:** `_lym/discussion/decision-record.md` D1 (line 54) against D6 (lines 146-149).
**Issue:** D1 says the runner runs "the same request in-process with `--wait` semantics, slot wait and tiers included". `lym gate card-verify` has no `--wait` flag and no tiers, and D6 runs its line through `cardverify.Runner`.

### [LOW] D3 names one lease-shortening write where there are two

**Location:** `_lym/discussion/decision-record.md` D3 (line 90) against D1 (line 50).
**Issue:** D3 calls the supersede expiry "the one write that moves it earlier", while D1's start failure also expires the lease.

### [NIT] The wall time leaves out the pre-tier phase

**Location:** `_lym/discussion/decision-record.md` D5 (line 133).
**Issue:** Records start at the first tier's `WaitStart` (`internal/gatecli/run.go:460`) or the build's `Start`, after config load, scope computation and the guard scan (`run.go:53-90`), which the front also blocks through.
D5's Bound already prices a mis-pick as cache money only, so only the claim that the span covers what the front blocks through is wrong.

## Rejected

- bound-breaker F7, "a session end that ends the slug's processes" names no real path: `lym tender pause` ends a held session's slug processes through `runproc.End` (`internal/tendercli/deps.go:113`), and the anchor end it adds is harmless, as existing-paths traced: the anchor move changes the key tree, so the old run reaches no caller.
- Handler candidate, D2 drops the capped slot wait for a call with no strand: D2 states the change and its bound (line 80 and 85), its cost is cache money for a session lym did not spawn, and the board's agent waits are all strand-carrying sessions.

## Focus areas settled

- Runner liveness: a SIGTERM through `runproc.End` either lets the runner write its marked exit file or, past the 5 s grace, falls to SIGKILL and D1's dead-process supersede. That holds on Linux for `lym gate test` runners. It breaks on Windows (F1) and for a card-verify runner's line (F3).
- Lease semantics: a shorter-cap front never moves the lease earlier, since it writes only a later deadline, and D3 orders the lease write before the spawn. Both hold; F9 is a wording gap.
- Median inputs: `gateslot.RunRecord` has no cancel mark today, as D5 states, and the record times D5 names are right. The mark misses build and blocked-tier records (F4), and `lym gate report` ignores it (F6).
- Card-verify path: a `--wait` line runs unslotted as a child under the caller's context, and any other line takes one slot under that context. Both cancel on a lease pass, but a killed runner orphans either kind (F3).
- Runner verb bound: a direct call reaches only a subdirectory holding a live, unclaimed lease, refuses otherwise naming `lym gate test` and `lym gate card-verify`, and adds at most a run its lease bounds. That holds, apart from the late-claim race (F5).
