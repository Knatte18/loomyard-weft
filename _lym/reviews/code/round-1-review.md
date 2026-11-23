---
verdict: BLOCKING
findings:
  - id: F1
    severity: BLOCKING
    class: consistency
    origin: assist:assist
    location: contracts/stencils/beam/beam-template-describe.md:43
    summary: The describer stencil restates the final-summary spec's format (the title-line syntax and its 72-character cap, the Merge danger section) instead of only pointing at it, which PATTERN-stencil-ownership forbids.
    fix: "In the stencil's `## What to write` list, drop the restated format: the `A first line # <title>, the title at most 72 characters` item and the description of the Merge danger section's door and blast radius. Keep the pointer to `loom-final-summary-spec.md` as the format contract, and keep only the beam-specific content rules: rewrite whole, state the end result and never the way there, list every member pull request by number, and call the Merge danger section for the beam's whole landing."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/dobbyengine/loop.go:717
    summary: A describeStep error returns from tick before tidyStep and landBeam, so a persistent spawn failure (unreadable facts, an unbuildable runner, a missing stencil) blocks the board tidy and the beam's landing step on every tick, retries the spawn every tick instead of at the next landing, and never tells the orch.
    fix: "In `daemon.tick` (internal/dobbyengine/loop.go), stop a describeStep error from returning ahead of tidyStep and landBeam: log it once per episode through a new `logger.Episode` on the daemon (as `describePullRequest` is) and go on. In `describeStep`/`spawnDescribe` (internal/dobbyengine/describe.go), on a spawn failure drop the landings through `endDescribe` and post one line to the nearest orch naming the cause with `describeStandsWayForward`, so the rewrite is retried at the next member landing as the board entry's Design asks. Update the spawn-failure table in TestDescribeStep to assert the landings are dropped and the line posted, and add a tick-level row asserting tidyStep still runs after a describe failure."
  - id: F3
    severity: MEDIUM
    class: design
    origin: assist:assist
    location: internal/dobbyengine/describe.go:225
    summary: settleDescribe returns on a draftFindings error before it checks the deadline, so a persistent read error on the draft (a directory or an unreadable file at DraftPath) keeps the attempt in flight forever, and no later landing ever starts a rewrite.
    fix: "In `settleDescribe`, when `draftFindings` fails, return the error only while `now.Before(describe.Deadline)`; past the deadline, stop the strand, call `endDescribe(describe.Landings)` and post the timeout line with the read error as its cause. Extend the TestDescribeStep subtest `a draft whose findings do not read fails the step and keeps the agent` to advance past the deadline and assert the attempt is cleared and a later landing starts a second agent."
  - id: F4
    severity: MEDIUM
    class: design
    origin: lens:code-affected-set
    location: internal/dobbyengine/loop.go:533
    summary: When a beam's worktree is removed, exitWorktreeGone stops only the kept board-tidy agent; a describer agent in flight is never kept or stopped, so its pane is left running against a removed worktree.
    fix: "Keep the describer in flight beside `keptTidy`: have `readKeptTidy`/`rememberKeptTidy` also keep `State.Describe` (for example a `keptDescribe *DescribeState` field), and in `exitWorktreeGone` stop its `StrandGUID` through `Deps.PrimeTidyShuttle` with the same prime-log lines as the tidy. Add a row to the existing worktree-gone test asserting the describer's strand is stopped."
  - id: F5
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/dobbyengine/describe.go:234
    summary: When the rewrite commits but the pull request update fails, recordDescription errors with 'recorded, but its pull request is not updated', and settleDescribe posts that the rewrite was not recorded and the old description stands, which misreports a committed description and hides the stale pull request.
    fix: "Let the Record seam tell the two outcomes apart: in internal/beamcli/dobbydeps.go `beamDescribeDeps`, make Record return whether the description was recorded beside the error (or a sentinel error wrapping the pr-update failure). In `settleDescribe`, word that case as the description recorded but the pull request not updated, with the way forward `lym beam pr update <slug>`, and keep the `describeStandsWayForward` line for failures before the record. Update the `BeamDescribeDeps.Record` doc comment and add a row to the TestDescribeStep refused-record subtest for the recorded-but-not-updated case."
  - id: F6
    severity: MEDIUM
    class: consistency
    origin: assist:assist
    location: "_lym/landing/summary.md#Merge danger"
    summary: The change description calls the door two-way with blast radius dobby, but the diff makes dobby commit the beam's description, push the beam and edit the pull request's body on GitHub on its own, which the rubric names as an outward push a two-way call cannot carry.
    fix: "In _lym/landing/summary.md's `## Merge danger`, call the door one-way, saying that reverting the code does not restore a pull request body dobby has already edited or undo the description commits it made, and widen the blast radius to dobby, the beam's change description and its pull request on GitHub."
  - id: F7
    severity: MEDIUM
    class: design
    origin: assist:assist
    location: internal/dobbyengine/describe_test.go:93
    summary: The describe tests call countLandings and describeStep directly and no tick test sets BeamDescribe, so the tick wiring is unpinned, including the hold that skips the description step under an anchor change, an adopt hold or a catch-up; deleting the describeStep call from tick fails no test.
    fix: "Add one row to the existing tick or scheduling-hold tests in internal/dobbyengine that sets `Deps.BeamDescribe` and `PullRequestRecorded`. Assert that a landing under a standing anchor-move record starts no describer, and that once the record is gone the next tick starts one through `tick`. Do not add a new helper-level test."
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/dobbyengine/describe.go:231
    summary: settleDescribe clears the attempt before d.post, and post only logs a failed Notice, so a failed post loses the one line the change description promises the orch; settleTidy posts before clearing so a failed post is retried.
    fix: "In `settleDescribe`, post the refused-record and timeout lines through `d.deps.Notice` before `endDescribe` clears the attempt, as `settleTidy` does, and return the post error so the next tick retries the line. Do not call Record again on that retry. For example, store the owed line in `DescribeState` once Record has run, and on a later settle post only that line. Extend the refused-record subtest with a failing Notice followed by a succeeding one."
  - id: F9
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/dobbyengine/describe.go:228
    summary: Dobby records the first draft that validates and stops the strand at once, so an agent that writes the draft and then refines it has an intermediate version committed and pushed to the pull request.
    fix: "Record the draft only once the agent has ended its turn or the shuttle run has settled: have the AgentShuttle report whether the strand's run ended (or check the run's gate outcome) and keep waiting while it runs and its deadline has not passed. Otherwise state in the stencil's Rules that the agent writes the draft once, whole, and never edits it after."
  - id: F10
    severity: LOW
    class: design
    origin: assist:assist
    location: internal/landingshed/beamdescribe.go:197
    summary: ReplaceBeamDescription writes the draft with os.WriteFile, which truncates summary.md first, so a write that fails partway leaves the recorded description damaged before the restore path is reached.
    fix: "In `ReplaceBeamDescription`, write the draft over `path` atomically: write a temp file in the same directory and rename it over `path`, through the repository's existing atomic-write helper if one exists. On a write failure the old description then stays intact and `commit` is not called. Keep the restore on a failed commit."
  - id: F11
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/dobbyengine/describe_test.go:148
    summary: Mutation survivor at describe.go:158; no test pins that `d.busy = d.busy || waiting` keeps a busy flag an earlier step set when the describer settles, so `d.busy = waiting` would pass.
    fix: "In the TestDescribeStep subtest `a passing draft is recorded once and ends the agent…`, set `h.d.busy = true` before the settling step and assert `h.d.busy` stays true afterwards."
  - id: F12
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/beamcli/dobbydeps_integration_test.go:569
    summary: The wiring test asserts only a non-nil Record and a positive Timeout, so nothing pins that the describer runs on landing.yaml's describe model and describe_timeout_min, or that its runnerAgentShuttle carries the describe gate.
    fix: "In TestHubDobbyDeps_PrimeTidyShuttle, assert `beamDeps.BeamDescribe.Timeout` equals the hub's `describe_timeout_min` minutes (30 at the shipped default) and `beamDeps.BeamDescribe.Model` resolves from its `describe` spec. Also assert that `beamDeps.BeamDescribe.Shuttle.(*runnerAgentShuttle).gate` is non-nil while `TidyShuttle`'s gate is nil."
  - id: F13
    severity: LOW
    class: design
    origin: lens:code-tests
    location: internal/dobbyengine/describe_test.go:155
    summary: spawnDescribe's documented contract that a state record that cannot be written ends the started agent again has no test row.
    fix: "Add a row to the TestDescribeStep spawn-failure table that makes the state write after Start fail. For example, have the fake shuttle's Start replace the state path with a directory. Assert that the strand is in `h.shuttle.stopped` and the error is returned."
  - id: F14
    severity: LOW
    class: consistency
    origin: lens:code-affected-set
    location: docs/overview.md:294
    summary: The module listing says internal/dobbyengine spawns the board-tidy agent, but the diff makes it spawn the describer agent too.
    fix: "In docs/overview.md's `internal/dobbyengine/` line, name both agents: `… and spawns the board-tidy agent and a beam's describer agent`."
---

Reviewed `cce04199925a2dc6eee25716d8112c0649d98d40..HEAD` (HEAD 9558b0da6), against the decision record, whose `## Goal` matches the board entry `lym board get` returns, with no added entries.
Every goal point has a change, and every change traces to one or is forced by one (code-scope lens).
The landed line drops its follow-up, and no stencil, orch role text or doc still asks the orch to describe after a landing.
The hand path (`lym beam describe` with `from` empty) keeps its checks and behavior.

### [BLOCKING] The describer stencil restates the final-summary format

**Location:** contracts/stencils/beam/beam-template-describe.md:43

**Issue:** PATTERN-stencil-ownership's background (`pattern/PATTERN-stencil-ownership.md:36-37`) says an instruction file that needs another producer's format points at its format contract and "never restates the format".
The stencil points at `loom-final-summary-spec.md` and then restates its title-line syntax, the 72-character cap and the Merge danger section.
The landing Describe stencil does the same, but that existing case does not license a new one, and the rubric makes every violation of a listed constraint BLOCKING.

### [MEDIUM] A describe-step error stops the tidy and the beam's landing step every tick

**Location:** internal/dobbyengine/loop.go:717

**Issue:** `tick` returns on a `describeStep` error before `tidyStep` and `landBeam`.
`spawnDescribe` keeps the landings on every failure, so a persistent cause makes every tick fail again.
Such causes include an origin record that does not read, a runner that does not build, an empty branch that fails the fill, and a missing stencil.
Each re-failure is logged only once, as a "tick failing" line (loop.go:571-576).
The beam's board tidy and its landing step stay held all that time.
The Design asks for a rewrite that fails to be retried at the next landing with the old description standing; this one is retried every tick, and the orch never hears of it.

### [MEDIUM] An unreadable draft keeps the attempt in flight forever

**Location:** internal/dobbyengine/describe.go:225

**Issue:** `settleDescribe` returns the `draftFindings` error before its deadline check, so the attempt is never ended or cleared.
`describeStep` sees `current.Describe` set on every later tick and never spawns another agent.
Every member landing after that goes undescribed, against the Design's retry at the next landing.
The tick also fails before `tidyStep`, as in F2.

### [MEDIUM] A removed worktree leaves the describer agent running

**Location:** internal/dobbyengine/loop.go:533

**Issue:** `exitWorktreeGone` stops only `d.keptTidy` (loop.go:505-551).
The describer's run is waited on by a goroutine in the dobby process, which ends with the dobby, so nothing stops that agent's pane.
The tidy agent is guarded against exactly this case through `PrimeTidyShuttle`.

### [MEDIUM] A failed pull-request update is reported as an unrecorded rewrite

**Location:** internal/dobbyengine/describe.go:234

**Issue:** `recordDescription` commits the description before `prUpdate`.
A failed push or GitHub edit then returns "the description of b is recorded, but its pull request is not updated" (internal/beamcli/describe.go:91).
`settleDescribe` wraps it as "was not recorded … the recorded description stands, and the next member landing rewrites it again".
That line contradicts the committed state and gives the orch the wrong way forward, since the pull request needs `lym beam pr update`.

### [MEDIUM] Merge danger is called two-way for a change that pushes outward

**Location:** _lym/landing/summary.md#Merge danger

**Issue:** The rubric makes a two-way call a finding when the diff pushes anything outward.
Dobby now runs `recordDescription` on its own, and that runs the `pr update` sequence (internal/beamcli/dobbydeps.go `beamDescribeDeps` Record → `rewriteDescription`), which pushes the beam branch and edits the pull request body.
A revert of the code does not restore a body dobby already edited.
The blast radius "dobby" also leaves out the beam's change description and its pull request.

### [MEDIUM] The tick wiring of the describe step is unpinned

**Location:** internal/dobbyengine/describe_test.go:93

**Issue:** The harness's `step` calls `countLandings` and `describeStep` directly.
No tick-level test sets `Deps.BeamDescribe`, since `describes()` is false in every other dobbyengine test.
So removing the `describeStep` call from `tick` fails no test, and neither does moving it out of the `scheduling` block.
The new tick doc comment states that hold, "runs no tidy or description step".
PATTERN-test-speed's background (line 247) wants behavior pinned at the surface that uses a helper.
That is a coverage gap for the hold, filed here at MEDIUM; see Rejected for why not BLOCKING.

### [LOW] A failed post loses the failure line

**Location:** internal/dobbyengine/describe.go:231

**Issue:** `endDescribe` clears the attempt before `d.post`, and `post` (loop.go:440-447) only logs a failed Notice.
No state is left from which a later tick could repost the line.
`settleTidy` posts first and returns the error, so its line is retried.

### [LOW] An intermediate draft can be recorded and published

**Location:** internal/dobbyengine/describe.go:228

**Issue:** `settleDescribe` records the first draft that passes the findings check and stops the strand, without regard to whether the agent has ended its turn.
An agent that writes a valid draft and then keeps editing it has its first version committed and sent to the pull request.

### [LOW] The draft write can truncate the recorded description

**Location:** internal/landingshed/beamdescribe.go:197

**Issue:** `os.WriteFile` truncates `summary.md` before it writes.
A write that fails partway returns "write the description" before the restore that only a failed commit triggers.
That breaks the `land.go` doc's claim that the description stands as it was when the draft is refused.

### [LOW] The busy flag's OR is unpinned

**Location:** internal/dobbyengine/describe_test.go:148

**Issue:** The mutation report's survivor at describe.go:158 deletes the `d.busy ||` clause.
The only assertion starts from `busy=false` with the agent waiting, so a settle that clears a busy flag an earlier step set goes unseen.
Survivors 166 (an episode log line) and 282 (a warn on a stop failure) change only logging, and no test could reasonably kill them.

### [LOW] The describer's wiring is asserted only loosely

**Location:** internal/beamcli/dobbydeps_integration_test.go:569

**Issue:** The test checks only `Record != nil` and `Timeout > 0`.
A wrong model, a wrong positive timeout, or a `runnerAgentShuttle` built without the describe gate passes it.
Without the gate, the describer loses the three re-prompts the change description claims.

### [LOW] The strand stop after a failed state write is untested

**Location:** internal/dobbyengine/describe_test.go:155

**Issue:** `spawnDescribe` stops the started strand when the state record cannot be written.
Its doc comment states that contract, and no row exercises it.

### [LOW] The overview's module line names only the tidy agent

**Location:** docs/overview.md:294

**Issue:** The `internal/dobbyengine/` entry says dobby "spawns the board-tidy agent".
The diff adds the describer agent, and only the beam section at line 837 is updated.

## Rejected

- code-tests 2: the landing a tidy in flight holds back is held by `countLandings`'s existing skip on `s.Tidy != nil`, which the tidy tests pin. The describe append sits in the same loop, so no new behavior is unpinned.
- code-tests 7: the non-ENOENT read of the recorded description is a one-line error pass-through, and the existing rows pin the error-wrapping shape.
- code-correctness F5, the DescribedAt stamp part: the commit closure in `hubLandDeps.RecordDescription` stamps before committing on the hand path too. That behavior predates the diff, which does not change it. The truncation part is kept as F10.
- assist F2 at BLOCKING: tests that drive the daemon's step methods directly are this package's existing practice; the tidy harness drives `countLandings` and `tidyStep` the same way. TestDescribeStep tests the step's behavior as one unit, not each helper. The real gap, the tick wiring and its hold, is kept as F7 at MEDIUM.
- assist F7 (the stencil omits reading every member pull request): the prompt lists each member pull request by number from its merge commit with its first body sentence, and has the agent read each member's commit message and the beam's diff. PATTERN-github-auth keeps GitHub reads inside `internal/githubclient`, so that is a sound reading of the board entry's "every member pull request", as the code-scope lens also judged.
