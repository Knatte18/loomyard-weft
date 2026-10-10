# Card 22 — webster-review-failure

**Create:**
- `plan:internal/loomshed#PublishFailureNote` -> `func PublishFailureNote(worktreeRoot, verifyDir string) string`

**Edit:**
- `internal/shedadapters#ReadRubric` -> `func ReadRubric(stencilsDir, name, specsDir, publishFailure string) (string, error)`
- `internal/shedadapters#BouncerConfig`
- `internal/shedadapters#Bouncer.runSeedSpawn`
- `internal/shedadapters#Bouncer.judgeCall`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#burlerRoundProfile` -> `func burlerRoundProfile(cfg Config, stencilsDir, specsDir, publishFailure string) (burlerengine.Profile, error)`
- `internal/shedrecipe#burlerRoundEntry`
- `internal/shedrecipe#bouncerEntry`
- `internal/loomcli#loomCLI.wire`
- `internal/shedadapters/rubric_test.go`
- `contracts/stencils/rubric_test.go`
- `contracts/stencils/claims_test.go`
- `contracts/stencils/loom/loom-rubric-webster-review.md`
- `internal/loomshed/publishfailure_test.go`

**Uses:**
- `plan:internal/loomshed#checkedPublishFailure`

**Intent:**
Make the Webster-Review round after a goto from a failed Publish review the failure and the merge that brought it (D12).
`PublishFailureNote` renders the checked record (the previous card's `checkedPublishFailure`) as markdown: the failing verify from the kind's closed set, the failing tests, the log path, and the merge-in commit with the command that reads what it brought in, `git diff <merge>^1 <merge>`; a field that failed its check is absent, and with no record it renders `none`.
`ReadRubric` fills a new optional `publish_failure` marker from its argument beside `specs_dir` and `stencils_dir`.
`shedrecipe.Env` gains `PublishFailure func() string`; `loomCLI.wire` sets it to `PublishFailureNote` over the worktree and the verify directory.
`burlerRoundEntry` calls it once and passes the note to `burlerRoundProfile`, which hands it to `ReadRubric`; each `lyx loom step` builds its producers afresh, so the round reads the record present when it steps.
`BouncerConfig` gains a `PublishFailure func() string` that `bouncerEntry` fills from the env, and the seed and judge calls pass its result to `ReadRubric` at call time; a nil function renders `none`.
`loom-rubric-webster-review.md` gains a Publish-failure section holding the marker: when it names a failure, the reviewer raises each failing test as a BLOCKING finding, reading the merge's diff and the log for its cause, in addition to the branch's own range; its range section, the single definition of the range, states that a rendered merge-in commit widens the range by that merge's diff and that nothing else does.
Discussion- and Plan-Review rubrics carry no such marker.
`contracts/stencils/rubric_test.go`'s marker allowlist gains `publish_failure` and its render rows pass a note; `internal/shedadapters/rubric_test.go` gains a row rendering a note and one rendering `none`; `claims_test.go` pins the rubric's BLOCKING sentence; `publishfailure_test.go` gains the note's rows, including a record whose dropped fields do not render.
Bound: the rendered record is the only widening; with no record the range rules stay as they are.

**ImpactSummary:** The rubric reader gains one parameter with three production callers and two test files; only the Webster-Review rubric carries the marker, so other reviews render unchanged.
