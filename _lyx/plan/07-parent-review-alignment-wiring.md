# Card 7 — parent-review-alignment-wiring

**Create:**
- `internal/loomcli/alignmentwiring_test.go`
- `plan:internal/loomcli#parentReviewAligner` -> `type parentReviewAligner struct`
- `plan:internal/loomcli#parentReviewAligner.Spawn` -> `func (p *parentReviewAligner) Spawn(round int) (parentreview.Alignment, error)`
- `plan:internal/loomcli#parentReviewAligner.Poll` -> `func (p *parentReviewAligner) Poll(a parentreview.Alignment) (parentreview.AlignmentPoll, error)`
- `plan:internal/loomcli#parentReviewAligner.Close` -> `func (p *parentReviewAligner) Close(a parentreview.Alignment, reason string) error`
- `plan:internal/loomcli#newParentReviewAligner` -> `func newParentReviewAligner(location *lyxcwd.Location, runID string, cfg loomengine.Config, registry modelspec.Registry, stencilsDir string, shuttle alignment.Shuttle, notes func() ([]boardengine.Task, error)) (parentreview.Aligner, error)`
- `plan:internal/loomcli#hubBoardTasks` -> `func hubBoardTasks(location *lyxcwd.Location) func() ([]boardengine.Task, error)`
- `plan:internal/loomcli#alignmentOff` -> `func alignmentOff(location *lyxcwd.Location, cfg loomengine.Config) bool`
- `plan:internal/loomcli#alignmentSettings` -> `func alignmentSettings(cfg loomengine.Config, registry modelspec.Registry) (alignment.Settings, error)`
- `plan:internal/loomcli#alignmentSnapshot` -> `func alignmentSnapshot(decisionRecord, slug string, notes func() ([]boardengine.Task, error)) func() (string, error)`
- `plan:internal/loomcli#TestParentReviewAligner` -> `func TestParentReviewAligner(t *testing.T)`

**Edit:**
- `internal/loomengine#ParentReviewBrief` -> `func ParentReviewBrief(stencilsDir, slug, decisionRecordPath, supportLogPath, alignmentReport, alignmentReason string) (string, error)`
- `internal/loomengine#TestParentReviewBrief_NamesPathsAndSubmitLines`
- `contracts/stencils/loom/loom-template-parent-review-brief.md`
- `internal/loomcli#newParentReviewConfig`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#statusCommitPathspec`
- `internal/loomcli#TestStatusCommitPathspec`
- `internal/loomcli/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/alignment#Session.Begin`
- `plan:internal/alignment#Session.Poll`
- `plan:internal/alignment#Session.Close`
- `plan:internal/parentreview#Aligner`

**Intent:**
Wire card 6's `Aligner` to card 5's session for a loom run, and make the fallback brief name the alignment report or why no verdict came (D4).

The brief:
- `loom-template-parent-review-brief.md` gains an alignment section with two top-level markers, `{{.alignment_report}}` and `{{.alignment_reason}}`. It says that when the report is not `none`, the alignment agent's report is to be read first, and that the reason says why the agent did not settle the review. Both read `none` when no agent ran. The leading comment lists the new markers.
- `ParentReviewBrief` takes `alignmentReport` and `alignmentReason` and fills them, with an empty value filled as `none`.
- Its two callers change: `newParentReviewConfig`'s `RenderBrief` passes two empty strings, and `TestParentReviewBrief_NamesPathsAndSubmitLines` passes empty values and gains a row with a report path and a reason, asserting both are named and no marker is left.
- `newParentReviewConfig` also sets `RenderAlignmentBrief` to the same render with the report and reason given.

The adapter, in a new file `alignmentwiring.go`:
- `alignmentOff` is true when `cfg.SkipAlignment` is set or the hub has no board dir (a stat of `fabricengine.BoardDir(location.HubPath)`), so both alignment points behave exactly as before this task (D6, D7).
- `alignmentSettings` resolves `loomengine.ResolveAlignment` into `alignment.Settings`. Card 10's PR-side wiring reuses it.
- `hubBoardTasks(location)` returns the closure `boardengine.OpenHub(location.HubPath).ListTasksFull`, the production board read.
- `alignmentSnapshot` returns a closure that reads the decision record's bytes and, through `notes`, the text (title, brief and body) of every `KindNote` task whose text names `slug`, sorted by id, and returns `alignment.Snapshot` over them. That is D5's decision snapshot, over D8's two places a decision on a held task is recorded. A record that does not exist yet reads as empty.
- `newParentReviewAligner` returns nil when `alignmentOff`. Otherwise it builds a `parentReviewAligner` over `alignment.NewSession`, with `Root` set to `shedrun.AlignmentDir(location, runID, "parent-review")`, the stencils dir, the told `shuttle` and `time.Now`. Its snapshot is `alignmentSnapshot(loomengine.DiscussionDecisionRecord(location), seedSlug(location.WorktreeName), notes)`. It carries the settings, the snapshot closure and a `PromptInput` for `alignment.PointParentReview`: slug, worktree, decision record and support log.
- `Spawn(round)` reads the snapshot and calls `Begin` with key `round-<N>` and the round's `brief.md` path under `loomengine.LoomParentReviewDir`. It returns `Alignment{Attempt, Strand, SpawnedAt}`.
- `Poll(a)` reads the attempt from `a.Attempt` and maps `Session.Poll`:
  - pending is not done.
  - A cause gives `Done` with the cause and detail as the reason, plus the report path when the file exists.
  - An `approve` whose snapshot differs from the attempt's is `escalate`, with the reason "a decision was recorded for this task while the alignment agent ran", so a decision added to the record or a board note naming the task during the review never lets the approve through.
  - Every other verdict passes through with the report path, and an `escalate` carries its question as the reason.
- `Close` calls `Session.Close`.

`loomCLI.wire` calls `newParentReviewAligner` after `newParentReviewConfig`, with `c.runID`, the loaded config, the registry, the stencils dir, `alignment.RunnerShuttle(runner)` and `hubBoardTasks(location)`, and sets `parentReviewCfg.Aligner` from it; an error is a wiring error.
`statusCommitPathspec` adds `shedrun.AlignmentRel(location, runID)` when `holdsFile(shedrun.AlignmentDir(location, runID, ""))` (an empty point joins to the `alignment` directory itself), beside the arbiter's, so the attempts, prompts, verdicts and reports are committed with the run records.
The function's doc comment, which lists every directory it stages, names the alignment directory beside the arbiter's.
`TestStatusCommitPathspec` gains a row with a file under the alignment directory.

`TestParentReviewAligner` is one table test over a fake `alignment.Shuttle` and a fake `notes` closure, on a temp location whose board dir is created or left out per row. Rows:
- `skip_alignment` gives a nil `Aligner`.
- No board dir gives nil.
- On, a `Spawn` twice for one round starts one agent.
- An approve with an unchanged snapshot polls `approve`.
- An approve after the decision record changed polls `escalate`, with the decision reason.
- An approve after the fake `notes` gains a note naming the slug does the same, while a note naming another slug leaves it `approve`.
- An escalate carries its question and report.

`doc.go` gains a paragraph on the parent-review alignment wiring, and `test-budget.yaml` raises `internal/loomcli` by one.

**ImpactSummary:** `ParentReviewBrief` gains two parameters, with both callers on this card; the loom wiring sets one more gate seam and one more status-commit path, and darn runs, which have no parent review, are untouched.
