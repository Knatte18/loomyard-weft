```yaml
lens: code-tests
outcome: reported
```

## Per-behaviour list

**dobbyengine**
- describeStep spawns the describer over the landings under a recorded pull request, with the prompt naming the draft, description, changes and branches: `TestDescribeStep` "a member landing … starts the describer".
- DescribeState is recorded (guid, draft, deadline, landings) and logged: same subtest.
- An agent in flight sets busy: same subtest, but only from busy=false. The `d.busy ||` clause is not pinned (mutation survivor at describe.go:158). **Finding 1.**
- Spawn failures (facts, nearest orch, engine, stencil) return the cause, record no agent and keep the landings: `TestDescribeStep` spawn-failure table.
- A spawn whose state write fails stops the strand again; a failed MkdirAll returns its error: none. **Finding 5.**
- An unreadable draft findings read fails the step and keeps the agent: `TestDescribeStep` "a draft whose findings do not read".
- Landings with no pull request recorded are dropped: `TestDescribeStep` "a landing with no pull request recorded".
- An unreadable landing record keeps the landings until it reads: `TestDescribeStep` "an unreadable landing record". Its once-per-episode log line (survivor 166) is log-only, so no finding.
- A passing draft is stopped, recorded once and logged, and a landing counted meanwhile gets its own rewrite: `TestDescribeStep` "a passing draft is recorded once". The stop failure being tolerated is pinned too; the warn branch (survivor 282) is log-only, so no finding.
- A refused record posts the exact line, logs it, clears the state, and the next landing retries: `TestDescribeStep` "a refused record posts why".
- At the deadline, no draft or a draft with findings ends the agent with an exact line and clears the state: `TestDescribeStep` deadline table.
- The prime's dobby rewrites nothing: `TestDescribeStep` "the prime's dobby rewrites nothing".
- countLandings appends to LandedSinceDescribe only when describes(): pinned indirectly by the subtests above and the prime subtest.
- A landing counted while a board-tidy agent is in flight is described after the tidy settles (doc.go bullet): none. **Finding 2.**
- tick runs describeStep only while scheduling, so not under an anchor change, adopt hold or catch-up (loop.go comment): none. No tick test sets BeamDescribe.Record. **Finding 3.**
- BeamDescribePrompt fills every marker, renders the parent and per-engine edit directives, and names empty changes: `TestBeamDescribePrompt`.
- BeamDescribeSpec sets engine, segment, config role, role, outputs and timeout: `TestBeamDescribePrompt`, plus the parity producer `buildBeamDescribe`.
- The landed line has no follow-up under a pull request: `TestLandNotices` "AMemberLandingUnderAPullRequestCarriesNoFollowUp". The no-commit and empty-diff lines keep rows `LandedWithNoCommitRecorded` and `EmptyLandingIsFlagged`.

**beamcli**
- judgeDescribable refactor, describeFailure and the describe verb record in place (from ""): `TestDescribe`, including the new recordedFrom assertion.
- rewriteDescription records from the draft and updates the pull request; a ready beam is refused with nothing recorded: `TestRewriteDescription`.
- RecordDescription with from set replaces the description through hubLandDeps: `TestHubLandDeps_DraftThenRecordStampsAndCommits`, tail.
- beamDescribeFacts returns the description path, branch, parent, and changes that exclude records-only commits: same integration test.
- describeGate builds one entry with name and attempts over OutputFiles[0], and passes or fails on the draft: `TestDescribeGate_JudgesTheSpecsOneOutputFile`.
- descriptionFindings: `TestDescriptionFindings`.
- hubDobbyDeps wires BeamDescribe on a beam and not on the prime: `TestHubDobbyDeps_PrimeTidyShuttle`, but only Record≠nil and Timeout&gt;0.
  - The model comes from landing.yaml `describe` and the timeout from `describe_timeout_min`: none. **Finding 4.**
  - The config fails to resolve, so the rewrite is off: none. **Finding 4.**
- runnerAgentShuttle.Start passes gate(spec) to StartGated; nil starts ungated: none. **Finding 6.**
- The renames resolveModel and runnerAgentShuttle: `TestHubDobbyDeps_PrimeTidyShuttle`.

**landingshed**
- ReplaceBeamDescription: a valid draft is committed; a finding or a missing draft leaves the description standing; a failed commit restores or removes: `TestReplaceBeamDescription`.
- ReplaceBeamDescription with an unreadable existing description (not ENOENT) refuses before any write: none. **Finding 7.**
- RenderBeamChanges, via beamDraft: `TestDraftBeamDescription` and `TestRecordBeamDescription`.

## Findings

1. **LOW · design** · `internal/dobbyengine/describe.go:158`
   - Summary: `d.busy = d.busy || waiting` is not pinned. Mutation survivor 158 shows that deleting `d.busy ||` passes, so a settle that clears a busy flag an earlier step set goes unseen.
   - Evidence: `describe_test.go:445-449` only steps from busy=false with the agent waiting.
   - Fix: add a row that sets busy=true before a step whose agent settles, and asserts busy stays true.

2. **LOW · design** · `internal/dobbyengine/tidy.go` countLandings, and the `doc.go` description-rewrite bullet "A landing that waits on a board-tidy agent in flight is described once the tidy settles"
   - Summary: no test drives a landing while `State.Tidy` is set and asserts it reaches LandedSinceDescribe, and a describer, once the tidy clears.
   - Evidence: describeHarness sets no TidyShuttle or Tidy state (`describe_test.go:345-387`), and no other dobbyengine test touches Describe (grep).

3. **LOW · design** · `internal/dobbyengine/loop.go:717-719`
   - Summary: the hold on the description step during an anchor change, adopt hold or catch-up (tick doc, "runs no tidy or description step") has no test.
   - Evidence: every tick test that reaches describeStep runs with BeamDescribe.Record nil, so describes() is false (no non-describe_test dobbyengine test references Describe).

4. **LOW · design** · `internal/beamcli/dobbydeps.go` beamDescribeDeps
   - Summary: nothing pins that the describer runs on landing.yaml's `describe` model and `describe_timeout_min`, or that a config that does not resolve leaves the rewrite off.
   - Evidence: `dobbydeps_integration_test.go:569-571` asserts only `Record != nil &amp;&amp; Timeout &gt; 0`. Any positive timeout or model passes.

5. **LOW · design** · `internal/dobbyengine/describe.go:210-216` spawnDescribe
   - Summary: a failed state write after Start stops the strand ("no unrecorded agent is left running"), and no test pins it. The MkdirAll error path is also unpinned.
   - Evidence: the spawn-failure table in `describe_test.go:452-503` breaks only facts, orch, engine and stencil.

6. **LOW · design** · `internal/beamcli/dobbydeps.go` runnerAgentShuttle.Start
   - Summary: no test pins that the describer's run is started gated through `gate(spec)` while the tidy's runs ungated. A Start that drops the gate passes every test, and the 3-attempt re-prompt is lost.
   - Evidence: `TestDescribeGate_JudgesTheSpecsOneOutputFile` calls describeGate directly (`dobbydeps_test.go:634`). `TestHubDobbyDeps_PrimeTidyShuttle` never inspects `BeamDescribe.Shuttle.(*runnerAgentShuttle).gate`.

7. **NIT · design** · `internal/landingshed/beamdescribe.go` ReplaceBeamDescription
   - Summary: the "read the recorded description" failure path (a non-ENOENT error before any write) has no row.
   - Evidence: the `TestReplaceBeamDescription` rows cover only an existing or absent file (`beamdescribe_test.go:156-169`).

## Tool calls

1. `grep` of `landnotice_test.go`, the other dobbyengine tests and `TestDraftBeamDescription`: to confirm the no-commit and empty-diff line rows still exist, that no other dobbyengine test exercises the describe step, and which rows the draft test holds. My context had only the diff hunks.
