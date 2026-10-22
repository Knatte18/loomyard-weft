---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    location: _lyx/discussion/decision-record.md:264
    summary: "The declared PATTERN-owner-machine text says a run is started, resumed or landed only on the machine that owns it, but D2 and D3 let any machine act on an entry with no owner"
    fix: "In the PATTERN changes section (line 264), reword the entry so it matches D2 and D3. Replace 'only on the machine that owns it' with 'only on a machine that does not read the entry as owned by another', and add that an entry with no owner recorded, and every entry of a hub with no prime owner recorded, reads as owned by no other machine. Keep every other clause of the entry as it is."
  - id: F2
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:108
    summary: "D5 step 1 reads a live pair from the reconcile report, but the reconcile asks LivePair only for a branch it would move, so the --drop-local live-pair refusal is unreachable for ahead, diverged, rewound or rewritten branches"
    fix: "In D5 step 1, state that adopt calls `LivePair` of the `OriginSeams` it built, itself and for every named slug the step would stop on local commits, instead of reading liveness from the report. The stop, the `--drop-local` refusal and the `--keep-local` record then all read that answer. Say that the reconcile asks `LivePair` only for a branch in state fast_forward or needs_replay. Reword the first acceptance criterion sentence of line 243 to match, and make clear a test reaches the refusal by handing `SeamOptions.HubPaused` a hub-only cover over a pair whose driver strand is alive under a slug-scope entry."
  - id: F3
    severity: MEDIUM
    class: design
    location: _lyx/discussion/decision-record.md:147
    summary: "D6 says the old machine's reconcile reads the slug ended and tears the pair down, and replays the step 2b records commit, but finalizeEntered keeps an ended run its own Finalize reached out of the ended set, so neither happens"
    fix: "In D6 lines 143 to 147, replace the claim that the old machine's reconcile reads the slug ended and tears the pair down with what finalizeEntered does. A Finalize history row, which a stopped Finalize leaves, makes finalizeEntered true, and the reset to origin's tip keeps parentHoldsTaskWork true, so the slug is dropped from the ended set. Then choose one of two. Either name the removal the halt note gives for the old machine's pair, such as `lyx fabric remove <slug>` after the reset, and say the step 2b records commit is replayed by a moving reconcile and not set aside, with that consequence in Open risks. Or decide that a run whose entry origin ended and whose parent push was refused is not exempt, with the code change to finalizeEntered and a test row for it. Adjust the line 252 acceptance criterion to the chosen behaviour."
  - id: F4
    severity: MEDIUM
    class: decision
    location: _lyx/discussion/decision-record.md:136
    summary: "D6 keeps the lock of a run whose Finalize is running and has merged, but names only parentHoldsTaskWork, which also holds after Finalize stops or the run is done, and names no signal for Finalize running"
    fix: "In D6 line 136, name the second signal InnerRun reads for 'Finalize is running', for example the child's inner shed status reading State running with CurrentProducer Finalize, which InnerRun already reads for its halt report. State the rule as that signal together with `parentHoldsTaskWork`, and add an acceptance row for a run stopped at Finalize with the parent already holding the work, whose lock is released."
  - id: F5
    severity: LOW
    class: decision
    location: _lyx/discussion/decision-record.md:254
    summary: "The re-entry criterion and D7 skip say a Finalize re-entered after its merge is not halted, but D3's Spawn check refuses an InnerRun re-entry of an owned-elsewhere run, and D7's unreachable-origin halt is not stated as skipped"
    fix: "In D7 and the line 254 criterion, say the skip covers the shed driver's in-process re-step after a transient parent-push failure, and that a re-entry through InnerRun's Spawn seam meets D3's refusal. Also decide whether the unreachable-origin halt of the landing reconcile is skipped for a Finalize past its parent-side merge, and write the answer next to the skip, with an acceptance row."
  - id: F6
    severity: LOW
    class: design
    location: _lyx/discussion/decision-record.md:143
    summary: "The halt note's `git reset --keep origin/<parent-branch>` drops every local parent commit origin lacks, not only the squash of the task, and the Decision states no bound on that"
    fix: "In D6 line 143 and the halt note rule on line 145, say the note names the reset together with listing the local parent commits origin lacks, and advises the reset only when the task's squash is the only such commit; otherwise it names moving that commit by hand. Add the same condition to the line 252 criterion."
---

### [BLOCKING] The declared entry says only the owner may act, the design lets an unowned entry be acted on

**Location:** `_lyx/discussion/decision-record.md:264`, against D2 line 48 to 50 and D3 line 67.
**Issue:** The declared `PATTERN-owner-machine` text reads "Starting, resuming or landing a run ... only on the machine that owns it".
D3 says "The shed resume verbs on a run-held task with no owner recorded proceed and write no owner", and D2 says that until a prime owner exists "no entry reads as owned elsewhere".
So on a machine that owns neither, a run is resumed and a start proceeds.
The design departs from the declared text, and the departure is not one of the declared exceptions.
The Constraints bullet at line 200 repeats the entry's wording and carries the same gap.
The other clauses of the entry agree with D3: the start's read after its pull, the re-read after an owner write's push and a resume with origin unreachable reading the local Bolt are each in D3.
The extra reads the Constraints bullet lists, adopt's `--all` selection and the status views, gate nothing and need no entry clause.

### [MEDIUM] Adopt cannot learn a live pair from the report for the non-moving states

**Location:** `_lyx/discussion/decision-record.md:108` and `:243`.
**Issue:** D5 step 1 stops a slug "whose pair it read live", and the criterion has `--drop-local` refuse a live pair.
In `internal/fabricengine/originreconcilemoves.go`, `move` returns at the `needsMove` test, which is true only for fast_forward and needs_replay, before it calls `pairLive`.
A branch judged ahead, diverged, rewound or rewritten never asks the `LivePair` seam, and no `BranchLive` is reported for it.
The adopt-back case the criterion describes, local commits made after a handover with the old driver's strands alive, is such a state for the code branch.
So the report does not say the pair is live, and `--drop-local` would move the branch under alive strands.
The seam itself is named correctly: `pairteardown.HubSeamOptions` fills `HubPaused` from `record.Covering(slug)`, which matches a slug-scope entry, and `SeamOptions.HubPaused` is the injection point a hub-only cover and a test reach.
The gap is who calls it.

### [MEDIUM] The old machine's pair and records commit after an adopter-first landing

**Location:** `_lyx/discussion/decision-record.md:147`, with `:144`.
**Issue:** D6 says that once origin marks the slug done "the old machine's reconcile reads it ended and tears its pair down as for any ended run".
`readOriginEnded` ends with `withoutFinalizeEntered`, and `finalizeEntered` in `internal/fabricengine/replay.go` is true on a Finalize history row, which D7 line 161 says a halted Finalize leaves.
The reset D6 advises leaves origin's parent holding the task's work, so `parentHoldsTaskWork` stays true too.
The slug is therefore never in the old machine's ended set: no teardown, and the step 2b records commit is not set aside as a commit of an ended run.
D6 line 144 says that commit is "replayed by the next moving reconcile", which is what happens, but it then publishes the old machine's run records for a run origin already ended, and D6 does not say so.
The weft part of the premise holds: `concludeMergeSides` skips the weft arm for every record this binary writes, so the merge itself makes no records commit and the parent's records branch needs no repair beyond that one commit.
The halt note's reset advice is conditioned on origin's parent holding the task's work, as the focus asks.

### [MEDIUM] "Finalize is running" has no detector

**Location:** `_lyx/discussion/decision-record.md:136`, with `:139`.
**Issue:** D6 and D7 name the same `parentHoldsTaskWork` for "already merged into the local parent".
D7 needs nothing else, since its check sits at Finalize's entry.
D6 adds "whose Finalize is running" and says the exemption ends when the child "stops at Finalize or reaches Done", but `parentHoldsTaskWork` is true in both of those states, and D6 names no second signal.
The plan writer must pick one, and a wrong pick keeps a lock the owner pause should hold, or releases one under a running Finalize.

### [LOW] Re-entry through InnerRun and the unreachable-origin halt

**Location:** `_lyx/discussion/decision-record.md:254`, with D3 line 60 and D7 line 176.
**Issue:** The skip in D7 holds for the re-step the shed driver makes in-process after a transient push failure, where `Finalize.Call` runs again without a spawn.
A re-entry through InnerRun's Spawn seam meets D3's refusal, which has no past-merge exemption, so the criterion's "re-entered after its parent-side merge" is wider than what holds.
The likely cause of a transient push failure is the network, and D7 halts Finalize at the landing reconcile when origin is unreachable; the decision does not say whether the past-merge skip covers that halt, which `Finalize` proceeds through today.

### [LOW] The advised reset drops more than the task's squash

**Location:** `_lyx/discussion/decision-record.md:143`.
**Issue:** `git reset --keep origin/<parent-branch>` moves the branch and discards every local commit origin lacks.
A local parent can hold another landing not yet pushed.
The record bounds the advice by whether origin's parent holds the task's work, but not by what else the reset removes.

## Focus verdicts

- **D7 skip and the shared detector:** the skip is specified and D6 and D7 both name `parentHoldsTaskWork`, exported from `internal/fabricengine` as the narrowest part of `finalizeEntered`; the in-process re-step passes. Open points are F4 and F5.
- **D5 step 1 and `LivePair`:** the Decision names the seam and the hub-only `HubPaused`; the stop is reachable by a test through `SeamOptions.HubPaused`. The open point is who asks `LivePair` (F2).
- **D6 parent records branch:** the weft is not a merge participant, so the warp-then-weft premise in the focus does not hold for this binary, and the halt note is conditioned on origin's parent holding the work. Open points are F3 and F6.
- **Constraints and the declared entry:** the start, post-push and offline-resume reads agree; the one disagreement is the unowned case (F1).
