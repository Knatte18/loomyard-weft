---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: design
    origin: lens:existing-paths
    location: _lym/discussion/decision-record.md D1 (join rule) and D3 (lease)
    summary: A front joins a run whose runner is dead, because the join checks only the lease, the delivered mark and the tree, so after a SIGKILL, crash or OOM kill every re-run renews the lease and answers 75 done false forever with no fresh run.
    fix: In D1, add to the join conditions that the runner recorded in the key directory is alive, checked by the pid and start time the runner writes into its subdirectory at start; a front that finds it gone starts a fresh runner under a new run id as for a tree change. In D3, state that a runner ended by a terminating signal writes its exit file with 128 plus the signal number before it exits, and add the bound that a runner killed without writing one is superseded at the next front. Add the case to D1's acceptance criteria.
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D3 (lease renewal), with D5 (long cap)
    summary: Each front overwrites the lease with its own cap plus grace, so a short-cap front joining after a long-cap front shortens the lease and the runner abandons with exit 1 while the long-cap front still waits, a false failure that breaks D5's bound of cache money only.
    fix: In D3, state that a joining front writes its deadline only when it is later than the current lease, so the lease is the latest deadline any front wrote, and that D1's supersede expiry is the one write that moves it earlier. Add an acceptance criterion that a short-cap front joining a long-cap front's run leaves the run going until the long-cap front's deadline.
  - id: F3
    severity: MEDIUM
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D5 (median filter)
    summary: D5 leaves out an abandoned call from the median, but gateslot.RunRecord has no field marking one and a tier the cancel kills still writes its record through execution.report, so a call abandoned in its last tier counts as full; D5 also does not say which record times form a call's wall time.
    fix: In D5, record the fact that today's RunRecord carries no abandonment mark and that a cancelled tier writes a record with its exit code (internal/gatecli/run.go report), then decide the mark, for example a new RunRecord field, abandoned (new), that the runner sets on every record of a call its lease or a supersede cancelled, and name it in the median's exclusion. State that a refused call writes no tier record and so drops out by itself. State that a call's wall time is the span from its earliest WaitStart to its latest End over the records sharing its call id.
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:walkthrough
    location: _lym/discussion/decision-record.md D6, with D1 (phase file, slot_free answer) and D3 (lease cancel)
    summary: A card Verify line is a --wait gate call that cardverify runs unslotted as a child process under context.Background, so a card-verify runner takes no slot, cannot know the phase or slot holders D1 reports, and has no context its lease cancel reaches.
    fix: Rewrite D6 to state what a card-verify runner does for each line kind, from internal/cardverify/runner.go - a --wait gate line runs as a child that takes its own slots, any other line takes one slot inside the runner - and what its phase file reports for each, such as running the line with no slot_free answer for a --wait gate line. State that the lease cancel kills the line's process tree, and say through which context or process-group kill. Drop the claim that the slot is always taken inside the runner.
  - id: F5
    severity: MEDIUM
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md D1 (hidden runner verb)
    summary: D1 adds a hidden internal-audience verb that starts a detached gate run, but states no bound on an agent or operator calling it directly, which would start a detached gate outside the gate rule's foreground requirement with no front writing a lease.
    fix: In D1, add a Bound for the runner verb that states what a direct call can do and what limits it - for example that it refuses unless the run subdirectory it is told holds a lease a front wrote, so a direct call ends at once, or that a shellguard row refuses it from a session - and name the refusal's way forward. Add an acceptance criterion for a direct call.
  - id: F6
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md D3 (first lease read) and D1 (runner start)
    summary: The record does not order the first lease write before the runner's spawn, nor say how the runner reads a missing lease, so the runner either abandons at once or runs unbounded if its front dies between the spawn and the write.
    fix: In D3, state that a front writes the lease into the new run's subdirectory before it spawns the runner, and that a runner finding no lease at its first read ends as abandoned.
  - id: F7
    severity: LOW
    class: design
    origin: lens:bound-breaker
    location: _lym/discussion/decision-record.md Constraints (Go gate calls carry --wait) and D2 Bound
    summary: The Constraint that every Go-run gate call carries --wait or an inherited slot is asserted for Go-rendered calls only; batten's Land row runs the operator's deploy command through verifyrun with the inherited environment and no slot, so a strand-carrying process running a deploy of lym gate test without --wait would detach and read 75 as a failed deploy.
    fix: In the Constraint, state how operator-written commands Go runs are covered - for example that every verifyrun caller strips agentname.StrandNameEnv from the command's environment as verifytree.Verify does - or record, with the evidence read, that no process running such a command carries a strand.
  - id: F8
    severity: LOW
    class: design
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D1 (result delivered) and D3 (several fronts)
    summary: When two fronts with the same words wait on one run, the first to read the exit file marks it delivered and the second starts a whole fresh run; the cost is unstated.
    fix: In D1's Bound, add that a second front with the same words that returns after the first front took the result starts a fresh run, which costs one more gate run and never a verdict.
  - id: F9
    severity: LOW
    class: consistency
    origin: lens:interactions
    location: _lym/discussion/decision-record.md D1 Bound
    summary: D1's Bound says the replay gives the same exit code as an in-process call, while the same Decision replays a recorded 75 as exit 1 with an added stderr line.
    fix: In D1's Bound, change the sentence to say the replay gives the same content and exit code as an in-process call except that a recorded 75 exits 1 with one added stderr line.
  - id: F10
    severity: LOW
    class: consistency
    origin: handler
    location: _lym/discussion/decision-record.md D9 third bullet and its acceptance criterion
    summary: D9 and its acceptance criterion change a 300000 ms Bash timeout in the stencils, but no stencil names 300000; the only sites are internal/shuttlecli/await.go's help and the shuttleengine.DefaultAwaitCap doc comment.
    fix: In D9's third bullet, replace in internal/shuttlecli/await.go's help and the stencils with in internal/shuttlecli/await.go's help and the shuttleengine.DefaultAwaitCap doc comment; in the D9 acceptance criterion, replace the stencils name a 3600000 ms timeout with the DefaultAwaitCap doc names it.
  - id: F11
    severity: LOW
    class: design
    origin: handler
    location: _lym/discussion/decision-record.md D4 Rationale (fork cache lifetime)
    summary: The 250 s template cap for every role rests on the fact that a fork runs at Claude Code's 5-minute subagent cache lifetime whatever CLAUDE_CODE_PROMPT_CACHE_TTL says, but the record names no source or Claude Code version for it.
    fix: In D4's Rationale, add the evidence for the fork lifetime, the Claude Code documentation or changelog entry and the version it holds for, or the observed transcript usage that shows a fork's cache write at the 5-minute price.
---

### [BLOCKING] A front joins a dead runner and answers 75 forever

**Location:** `_lym/discussion/decision-record.md` D1 join rule, D3 lease
**Issue:** D1 lets a front join while the lease holds, the result is not delivered and the tree matches.
Nothing checks that the runner process is still alive.
`runproc.End` sends SIGTERM and then SIGKILL after 5 s (`internal/runproc/end.go:19`), and is called on the slug from `internal/tendercli/deps.go:321` and `internal/pairteardown/teardown.go:603`; a crash, OOM kill or quick reboot has the same effect.
A runner killed that way writes no exit file.
Every front then renews the lease (D3), waits its cap on an exit file that never comes, and answers 75 `done: false`.
The implementer stencil reads 75 as "run again, never a failure", so the agent loops without end and no fresh run ever starts.
The record states no bound for this path.

### [MEDIUM] A later front can shorten the lease and abandon a run another front still waits on

**Location:** `_lym/discussion/decision-record.md` D3, D5
**Issue:** D3 says each front writes "its own cap deadline plus a grace of 120 s".
Front A, on D5's long cap, writes about 57 minutes.
Front B, with the same words and a 250 s cap, writes about 6 minutes.
B can be another role, or a call made after the median dropped below the break-even.
The runner then reads a passed lease, kills its tiers and records exit 1, "no caller came back", while A still blocks.
A replays a failure no test produced, which breaks D5's "never a verdict".

### [MEDIUM] The median cannot leave out an abandoned call from the records it reads

**Location:** `_lym/discussion/decision-record.md` D5
**Issue:** `gateslot.RunRecord` (`internal/gateslot/runlog.go:38-73`) has no field that marks an abandoned or cancelled call.
`execution.report` (`internal/gatecli/run.go:479-497`) writes a cancelled tier's record with its exit code before it halts.
A call abandoned in its last listed tier therefore has a record for every listed tier, with a short wall time, and enters the median as a full call.
The abandonment lives only in the runner's exit file, which `ReadRunRecords` does not read.
D5 also does not name the record times that make up a call's wall time.
`WallSec` leaves out the slot wait the front blocks through, and the span from `WaitStart` to `End` includes it.
Both are facts the design rests on, and the record leaves them for Plan to find out.

### [MEDIUM] The card-verify runner has no slot, phase or cancel path as D6 states it

**Location:** `_lym/discussion/decision-record.md` D6, D1, D3
**Issue:** D7 keeps a card's `**Verify:**` line as a `--wait` gate call.
`cardverify.Runner` runs such a line unslotted, as a child process under `context.Background()` (`internal/cardverify/runner.go:46-49`, `138`).
The line's slot wait and tiers happen inside that nested process.
So D6's "slot taken inside the runner" is false for it, D1's phase file and `slot_free: false` answer have no source, and D3's lease cancel has no context that reaches the line.

### [MEDIUM] The hidden runner verb is a new edge with no stated bound

**Location:** `_lym/discussion/decision-record.md` D1
**Issue:** D1 adds a hidden internal-audience verb under `lym gate` that runs a gate detached from any caller.
The rubric's attack-surface rule requires the Decision to state what such a verb can now skip and what bounds it.
D1 states neither for a direct call.
A session that runs the verb itself starts a detached gate run, which the gate rule ("never in the background") and the board ("an agent may not start a gate in the background") rule out.
No front has written a lease for that run, and D3 does not say how the runner reads a missing lease (F6).

### [LOW] The first lease write is not ordered before the runner's spawn

**Location:** `_lym/discussion/decision-record.md` D3
**Issue:** The runner reads the lease from its start.
If its front spawns it before writing the lease, the first read finds none.
Depending on how a missing lease is read, the runner either abandons at once or runs with no bound when its front dies between the spawn and the write.

### [LOW] The Go `--wait` Constraint does not cover operator-written commands Go runs

**Location:** `_lym/discussion/decision-record.md` Constraints, D2 Bound
**Issue:** `internal/battencli/land.go` `runDeploy` runs the landing config's deploy command through `verifyrun` with the inherited environment and holds no slot.
`verifytree.Verify` strips the strand (`internal/verifytree/verify.go:382`); this path does not.
Suppose a process that carries a strand runs a deploy command of `lym gate test` without `--wait`.
D2 detaches that call, and the Land row reads the 75 as a failed deploy.
That breaks the D2 Bound "never what runs or its verdict".
No reviewer confirmed whether the process that runs the Land row carries a strand, so this stays LOW.

### [LOW] A second front with the same words reruns the whole gate

**Location:** `_lym/discussion/decision-record.md` D1, D3
**Issue:** D3 expects several fronts to share one lease.
D1 gives the result to the first front that reads it and refuses the join to every later one.
A second front with the same words, such as another fork of the same session, therefore starts a full fresh run.
The cost is not stated in D1's Bound.

### [LOW] D1's replay Bound contradicts its own 75 rule

**Location:** `_lym/discussion/decision-record.md` D1 Bound
**Issue:** The Bound says the replay gives the same exit code as an in-process call.
The exit-file bullet replays a recorded 75 as exit 1 with an added line.

### [LOW] D9 names stencils that hold no 300000 ms timeout

**Location:** `_lym/discussion/decision-record.md` D9, acceptance criterion D9
**Issue:** A search of the hub's stencils and `contracts/` finds no 300000.
The text sites are `internal/shuttlecli/await.go`'s help and the `shuttleengine.DefaultAwaitCap` doc comment (`internal/shuttleengine/await.go:13`).
The acceptance criterion therefore has no stencil to check.

### [LOW] The fork lifetime the template cap rests on has no recorded evidence

**Location:** `_lym/discussion/decision-record.md` D4 Rationale
**Issue:** The writer stencil requires every fact the design rests on to be recorded with the evidence for it, named by source and version.
D4's choice of 250 s for every role rests on forks running at a 5-minute cache lifetime even under a 1-hour session TTL.
D4 names no source for that fact.

## Rejected

- D4 numbered-role mismatch between the cap resolver and the lifetime lookup: rejected. `internal/reedengine/strand.go:243` numbers only the strand name, and `Spec.Role` stays the base role that `resolvePromptCacheTTL` reads, so stripping the base-N suffix matches it.
- D2 nested call under card-verify detaching: rejected. A `--wait` line runs in-process by its flag, and any other line holds a slot and passes `gateslot.InheritEnv` (`internal/cardverify/runner.go:75`).
- D1 new subdirectories under the gate wait directory confusing its readers: rejected. `gateslot.ReadWaits` reads only its own file glob.
- Auto-mode assumption lines restating Decisions: not a finding. That section names the self-picks by definition, and none of its lines carries a requirement the Decisions lack.
- Board-entry edits and task filings: none. No Decision edits this task's entry or files or promotes a task.
