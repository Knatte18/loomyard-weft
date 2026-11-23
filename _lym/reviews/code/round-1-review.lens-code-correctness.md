```yaml
lens: code-correctness
outcome: reported
```

**Per function or branch: the case I ran, and the result**
- `daemon.describes`: the prime (Slug "") gives false; a beam with Record and PullRequestRecorded set gives true. Correct.
- `daemon.countLandings`, the describes branch: a fresh landing m1 is appended to both lists. With a tidy in flight nothing is counted. With no pull request recorded the landing is appended and later dropped by describeStep. Correct.
- `daemon.describeStep`:
  - ReadState error: returns the error. Correct, but the error aborts the tick (see F1).
  - Describe in flight: settles, and busy is ORed in. Correct.
  - Empty LandedSinceDescribe: nil. Correct.
  - PullRequestRecorded error: logged once per episode, landings kept, nil returned. Correct.
  - No pull request: endDescribe drops the landings. Correct.
- `daemon.spawnDescribe`:
  - Facts, NearestOrch, MkdirAll, EngineName, prompt or Start fails: returns the error and keeps the landings. The landings are kept correctly, but the error is wrong in effect (F1).
  - UpdateState fails after Start: the strand is stopped and the error returned. Correct.
- `daemon.settleDescribe`:
  - Valid draft, Record ok: strand stopped, landings dropped, log line. Correct.
  - Valid draft, Record fails in the commit or the ready check: the line is posted and the landings dropped. Correct.
  - Valid draft, the commit succeeds but prUpdate fails: wrong message (F2).
  - Valid draft, endDescribe fails: recordErr is dropped unposted. Next tick the draft is re-recorded and the sequence repeats; tolerable.
  - Post fails: the line is lost (F3).
  - Before the deadline: waiting. Correct.
  - Past the deadline with no draft, or with a draft that has findings: stop, end, post with the cause. Correct.
  - An agent still writing when a valid draft first appears: recorded early (F4).
- `daemon.draftFindings`:
  - Missing file: not written. Correct.
  - Stat error: returned, and the tick aborts (part of F1).
  - Findings error: returned with the agent kept. Correct.
- `daemon.endDescribe`: Landings [m1], list [m1, m2] leaves [m2]. Correct.
- `daemon.stopDescribeStrand`: a stop error is logged. Correct.
- `BeamDescribePrompt`: empty Changes gets the placeholder, an empty Parent goes through parentdirective, a stencil read error is wrapped. An empty Branch (detached HEAD) makes Fill fail, which reaches F1. Otherwise correct.
- `BeamDescribeSpec`: fields mapped. Correct.
- `ReplaceBeamDescription`:
  - Missing draft: CheckMissing finding, refused, file untouched. Correct.
  - Draft with findings: refused, untouched. Correct.
  - Path absent, commit fails: the file is removed. Correct.
  - Path present, commit fails: the previous text is restored. Correct for summary.md, but the landing record's DescribedAt stamp is left (F5).
  - WriteFile fails partway: the truncated file is not restored (F5).
- `RenderBeamChanges`: nil gives "", and beamDraft's section is skipped. Correct.
- `hubLandDeps.RecordDescription`: from "" goes to Record and from set goes to Replace. Correct.
- `describe`, `rewriteDescription`, `judgeDescribable`, `recordDescription`, `describeFailure`: a ready beam is refused before anything is recorded. A way forward that is already present is kept. The behaviour matches the pre-refactor describe. Correct.
- `descriptionFindings`: missing file gives a finding, valid gives none. Correct.
- `beamDescribeDeps`: a config or model error returns an error, and the wiring logs it and leaves the rewrite off. Correct.
- `beamDescribeFacts`: no origin record gives an error with a way forward. Open or branch errors are passed through. Correct.
- `describeGate`: one OutputFile, judged. Correct.
- `runnerAgentShuttle.Start`: nil gate starts ungated (the tidy), a set gate starts gated. Correct.
- `rememberRuns` and `landNoticeLine` without the follow-up: on, off and never-read cases. Correct.

**Findings**

F1. MEDIUM, design: `internal/dobbyengine/loop.go:717-719` with `describe.go:188-209` and `describe.go:230-232`.
A describeStep error returns from tick before tidyStep and landBeam.
A spawn failure that persists (Facts or origin unreadable, a detached branch failing Fill, a runner that does not build, a missing stencil) keeps the landings. It therefore re-fails every tick, with only the once-per-episode "tick failing" log line. Meanwhile the board tidy and the beam's own landing step never run.
The rewrite is not retried at the next landing but every tick, and no line reaches the orch.
Evidence: loop.go tick, `if err := d.describeStep(ctx, now); err != nil { return err }` ahead of tidyStep and landBeam; spawnDescribe's error returns; the run loop's error handling at loop.go:571-576.

F2. LOW, design: `internal/dobbyengine/describe.go:240-243` with `internal/beamcli/describe.go:89-92`.
When the record commits but `prUpdate` fails, recordDescription returns "the description of b is recorded, but its pull request is not updated". settleDescribe then posts "...was not recorded: ...; the recorded description stands". That misreports a committed rewrite as unrecorded and hides that the pull request body is stale.

F3. LOW, design: `internal/dobbyengine/describe.go:237-243` and `describe.go:252-260`.
endDescribe clears the state before `d.post`, and `post` (loop.go:440-447) only logs a failed Notice. A failed post loses the one line the design promises the orch, unlike settleTidy, which posts before clearing so a failed post is retried.

F4. LOW, design: `internal/dobbyengine/describe.go:234-236`.
Dobby records the first version of the draft that validates and stops the strand, without waiting for the agent to end its turn. An agent that writes the draft and then refines it (Write, then Edit) has an intermediate version committed and pushed to the pull request.

F5. LOW, design: `internal/landingshed/beamdescribe.go:200-213` with `internal/beamcli/deps.go` RecordDescription's commit closure.
On a failed commit, ReplaceBeamDescription restores summary.md, but the closure has already stamped the landing record's DescribedAt, and nothing puts that back.
A failed `os.WriteFile(path, draft)` after truncation leaves the recorded description damaged, with no restore. land.go's doc claims "The description stands as it was when the draft is refused or the commit fails".

Tool calls:
1. `sed`/`grep` of `internal/beamcli/deps.go:305-345` and `RecordBeamDescription`: the diff showed only part of the RecordDescription closure (the stamp before the commit).
2. `grep` of `CommitAnchoredPathsWithTrailers`: to learn whether the commit helper can fail after committing; this hit the trailer wrapper.
3. `grep` of `CommitAnchoredPaths`: confirmed the commit does not push, so there is no post-commit failure path.
