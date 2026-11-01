---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:118
    summary: "D6 derives the fix-slug number from the board and the run records, but lym batten cleanup removes a landed run's record while its task branch stays, so a refiled slug meets the leftover-branch refusal."
    fix: "In D6's slug paragraph, add the task branches named `<prefix>-t<N>`, local and on origin, as a third source of the highest N, beside the board and the run records. Cut the claim that no earlier run or entry of the subject held the slug, or restate it as true over the three sources. Add a case to the D6 acceptance criterion on a second red check, a landed first fix task whose run directory `lym batten cleanup --apply` removed, which must file a slug with a higher N."
  - id: F2
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:123
    summary: "D6 removes the subject's own fix task before a filing only when it stands draft or halted as blocked or failed; a hand adopt whose check is red while it stands running, paused or awaiting files none and the record gives no notice or way forward."
    fix: "In D6, add a bullet beside the open-front-task bullet. While the subject's own fix task stands open with a run status whose state is running, paused or awaiting, the red check files none and removes none, and its notice names that task, its state and the hand step (`lym tender resume <slug>` for paused, `lym batten cancel <slug> --remove` then `lym board remove <slug>` to drop it), and says its landing starts the next check. Add one acceptance criterion under D6 for it."
  - id: F3
    severity: MEDIUM
    class: design
    location: _lym/discussion/decision-record.md:116
    summary: "D6 and D5 add to and reset the fix-try count but never say where it is stored; runCheck writes a fresh CheckRecord at its start, so a count kept there is wiped and the stated bound of adopt_fix_tries filings no longer holds."
    fix: "In D4 or D6, name the store of the fix-try count. Put it in the subject's holds file, or in a tries file beside it, below `fabricengine.AdoptScratchDir`, written through `internal/state` under its own lock. State that the check record's writes never touch it, that a filing adds one to it before the task is released, and that only a hand `lym adopt` resets it. Add the case to the D6 acceptance criterion on the try count."
  - id: F4
    severity: LOW
    class: consistency
    location: _lym/discussion/decision-record.md:144
    summary: "D7 and D6 define a halted run as a run record in state blocked or failed, but internal/shedengine/status.go says awaiting behaves like blocked wherever a halt is observed, so an awaiting run reads running_here."
    fix: "In D7's `halted` state, and in D6's halted-fix-task clause, read the run record state as blocked, failed or awaiting. Add awaiting to the `halted` row of the D7 acceptance criterion."
  - id: F5
    severity: LOW
    class: consistency
    location: _lym/discussion/decision-record.md:227
    summary: "The D3 acceptance criterion says no verb releases a hold by hand other than lym tender resume and that held runs start only at a green check, which contradicts D4's bound that lym batten cancel lifts a run's hold and that tender resume lets a run go on without the check."
    fix: "Reword the D3 acceptance bullet to match D4's bound. Held runs start by themselves only at a green check of their subject. The only hand lifts are `lym tender resume`, bare or by slug, and `lym batten cancel <slug>`, which lifts the holds of the run it ends; no release verb and no reed button exists."
---

### [MEDIUM] Fix-slug number is lost when cleanup removes the landed run record

**Location:** `_lym/discussion/decision-record.md:118`
**Issue:** D6 takes N as one more than the highest `<prefix>-t<N>` among board entries and the `shedrun` run records, and concludes that no filing meets the done-slug start refusal.
Land prunes the board entry, so after a fix lands only the run record carries its slug.
`cleanupSlug` and `removeRunRecords` in `internal/battencli/cleanup.go` remove the run directory of every done run; the file says it touches no branch.
`doneSlugRefusal` in `internal/battencli/arm.go` says a torn-down pair keeps its task branch locally and on the remote.
After `lym batten cleanup --apply`, both stores lose the slug, so the next red check refiles the same `-t<N>` and meets the create row's leftover-branch refusal.
The record's guarantee that each filing has a slug no earlier run held is false in that sequence.
The first-landed-then-second-try path the focus names is otherwise settled: the slug is new, the refusal is keyed on the slug, and the record states the mechanism.

### [MEDIUM] A red check over a running, paused or awaiting own fix task has no stated outcome

**Location:** `_lym/discussion/decision-record.md:123`
**Issue:** D8 files no front task while one stands open, and an entry with any run status is open.
D6 removes the subject's own fix task before a filing only when it is draft, or halted as state `blocked` or `failed`.
The "other front task" bullet excludes the fix task itself.
A hand `lym adopt` whose check is red while the fix task runs, is paused by the operator or awaits therefore reaches a filing that D8 forbids, with no removal and no notice defined.
The subject is not stuck for ever, since a landing runs the next check and a hand removal exists, but the operator gets no stated way forward.

### [MEDIUM] The fix-try count has no named store

**Location:** `_lym/discussion/decision-record.md:116`
**Issue:** D4 names the holds file as the store of held slugs, and says why the check record cannot be: `runCheck` in `internal/adoptcli/check.go` writes a fresh `CheckRecord` at its start and rewrites the whole record at each save and at its end.
The fix-try count has the same lifetime: it must survive every check, grow at each filing, reset at a hand adopt and not reset at a check started by a landing.
D5 and D6 say "resets" and "adds one" but never name where it lives.
A plan writer who keeps it in the check record loses it at each check start, and the Bound that at most `adopt_fix_tries` fix tasks are filed between two hand adopts then fails as a filing loop.
The record also does not say how the check tells a landing-started check from a hand-started one.

### [LOW] `awaiting` reads as running here

**Location:** `_lym/discussion/decision-record.md:144`
**Issue:** `internal/shedengine/status.go` lists six states and says `StateAwaiting` behaves like blocked wherever a halt is observed.
D7's `halted` takes only `blocked` and `failed`, so an awaiting run reads `running_here`, which tells the operator a stopped run is going.
D6's removal of a halted fix task uses the same two states.

### [LOW] D3 acceptance criterion disagrees with D4's lifters

**Location:** `_lym/discussion/decision-record.md:227`
**Issue:** The criterion first says no verb releases a hold by hand other than `lym tender resume`, then names `lym batten cancel` as a lifter limited to the run it ends, then says held runs start only at a green check.
D4's Bound says `lym tender resume`, bare or by slug, lets the run go on without the check, and `lym batten cancel` lifts every live entry of the slug's own scope (`liftSlugPauses` in `internal/battencli/cancelcmd.go`).
The `PATTERN-adopt-hold` text in the PATTERN changes section agrees with D4, so only the criterion is off.

## Checked and not a defect

- Hold lifters: D4, the D4 Bound and the declared `PATTERN-adopt-hold` text agree with `liftSlugPauses`, which lifts every live entry of the slug's own scope.
- Owner bound on the land action: the dobby loop in `internal/dobbyengine/loop.go` (`beamOwnedElsewhere`) exits on a probe tick when another machine owns the beam, so a second machine's dobby does not tick on; D8's owner test on recording a front entry, on running its action and at `lym adopt check` is an added bound on the window between probes, and is stated.
- Held-slug storage: the separate holds file is untouched by `runCheck`'s writes, and the green end and the subject-start rule read it.
- Front-task filing: `applyDraftRule` makes a create draft and a null status write the release, which is the create-then-release pair D8 states, and the dead-between-writes case is covered by D6 and D5.
- Front-task removal: `refuseNothingLeft` is met only when the cancel step runs, and D8 runs it only with a claim, record or pair and counts the nothing-left refusal as done; a failed removal stops the filing and names its hand step.
- Machine-failure stop: `internal/loomshed/darnwrite.go` appends `darnStopWayForward` after the stop report; D9's second marker removes it for a stop on the body's condition, and nothing further down the chain adds `lym loom resume`.
- The `light` profile's darn `code_max_bounces` of 1 is the value in `internal/runprofile/template.yaml`.
