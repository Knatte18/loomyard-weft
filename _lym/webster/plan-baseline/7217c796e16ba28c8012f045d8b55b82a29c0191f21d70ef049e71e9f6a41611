# Card 8 — loom-pr-findings-verb

**Create:**
- `plan:internal/loomcli#prFindingsDeps` -> `type prFindingsDeps struct`
- `plan:internal/loomcli#prFindingsVerb` -> `func prFindingsVerb(out io.Writer, file string, deps prFindingsDeps) int`
- `plan:internal/loomcli#sendBackEdge` -> `func sendBackEdge(producers []shedengine.ProducerDef, row string) (bool, string, error)`
- `plan:internal/loomcli#writeSendBackNote` -> `func writeSendBackNote(frictionDir string, n int, findingsPath string, findings []burlerengine.Finding) error`
- `plan:internal/loomcli#loomCLI.prFindingsCmd` -> `func (c *loomCLI) prFindingsCmd() *cobra.Command`
- `plan:internal/loomcli#prReviewSmallMax` -> `func prReviewSmallMax(l *lymcwd.Location) func() (int, error)`
- `plan:internal/loomcli#TestPRFindingsVerb` -> `func TestPRFindingsVerb(t *testing.T)`
- `plan:internal/loomcli#TestPRReviewSmallMax_ReadsThePairsLayers` -> `func TestPRReviewSmallMax_ReadsThePairsLayers(t *testing.T)`

**Edit:**
- `internal/loomcli#Command`

**Extends:**
- `//internal/loomcli/doc.go`
- `//docs/overview.md`

**Uses:**
- `internal/loomcli#pullRequestRecordCommitter`
- `internal/loomcli#loomCLI.prContinueCmd`
- `internal/loomcli#samplerDoneVerb`
- `internal/burlerengine#ParseReview`
- `internal/friction#NotePath`
- `internal/loomrecipe#New`
- `internal/loomrecipe#EngineRow`
- `internal/loomengine#LoomScratchDir`
- `internal/loomengine#LoomFrictionDir`
- `internal/pullrequest#Record`
- `internal/verifytree#DirtyPaths`
- `internal/loomcli#TestPullRequestDeps_ParentReviewWantedReadsThePairsLayers`

**Intent:** D4's verb `lym loom pr-findings <file>`, run by the PR-Review agent on every findings set, and D6's friction note.

`prFindingsDeps` carries the seams the verb reads: the task worktree's dirty paths, the record path and lock, the loom scratch, friction and stencils directories, `smallMax func() (int, error)`, the run's `pr_review_small_max`, `edge func() (bool, string, error)`, which answers whether the send-back edge can fire and why not, and `commit func() error`, the record's commit.
The command wires `smallMax` with `prReviewSmallMax(location)`, which reads `loomengine.LoadConfig(fabricengine.ConfigDirs(l), "loom")` at each call, so the task's `run_settings` reach it; `dirty` with `verifytree.DirtyPaths`; `edge` with `sendBackEdge` over the producers `loomrecipe.New` builds for the run's recipe and env, with the PR-Review row named by `loomrecipe.EngineRow`; and `commit` with `pullRequestRecordCommitter`, as `pr-continue` does.
It runs under the full wiring, not the lightweight one.
Its help text tells the agent to write the findings file under the task worktree's `.scratch/` directory, which git ignores, so the file is never one of the worktree's uncommitted paths.

`sendBackEdge` decides from the built rows and the run's settings, never the recipe name (`PATTERN-shed-recipe-registry`).
It answers false, with the reason, when the PR-Review row declares no `OnReroute`, when the target row's `EntryGate` does not hold, and when the target row's `MaxBounces` is 0, the Code-Review budget of 0 that stamps `zero_rounds` on Code-Bouncer so it approves without review.

`prFindingsVerb`, in order, each refusal with its way forward (`PATTERN-refusal-way-forward`):

1. A dirty worktree is refused: `loom: pr-findings: the task worktree has uncommitted paths: <paths>; way forward: commit or discard them, keeping the findings file under the task worktree's .scratch/ directory, which git ignores, then run lym loom pr-findings <file> again`.
2. A file `burlerengine.ParseReview` refuses is refused, naming the parser's error: `…; way forward: write the findings in burler's review format, its verdict line and findings as <stencilsDir>/burler/burler-review-format.md defines, then run lym loom pr-findings <file> again`, the path spelled from the told stencils directory, the hub's seeded copy the agent can read.
3. A run with no pull-request record is refused: `loom: pr-findings: this run has no pull request record, so there is no review to send findings from; way forward: wait until the PR-Review row has opened the pull request, then run lym loom pr-findings <file> again`.
4. A failed read of `smallMax` or `edge` is an error envelope naming what failed, worded as `samplerDoneReadFailure` is, with `way forward: run lym loom pr-findings <file> again, and tell the operator when it keeps failing`; the record is left unchanged.
5. A record already holding a send-back mark answers the send-back again, naming the stored file and writing nothing.
6. Every finding counts, whatever its severity.
   It sends back when the edge can fire and a finding is BLOCKING or the findings plus the record's `FixHereCount` exceed `smallMax`.
   It sets `n` to `SendBacks + 1`, copies the file to `prFindingsFile(scratchDir, n)`, writes the friction note, and writes `SendBack` and `SendBacks` into the record.
   It answers `send_back: true`, the file, and a note telling the agent to end its turn while the run sends the findings back to Code-Review.
7. Otherwise it answers fix-here: it adds the findings to `FixHereCount` and the file's sha256 to `CountedSets`, unless `CountedSets` already holds it, and the note tells the agent to fix them itself, naming the edge's reason when the edge cannot fire.

After a record write, a failed `commit` answers `loom: pr-findings: commit the pull request record: <error>; way forward: run lym loom pr-findings <file> again, and tell the operator when it keeps failing`.
That re-run progresses: a send-back meets its own mark (step 5) and commits, and a fix-here meets its own hash in `CountedSets` and counts nothing twice.

`writeSendBackNote` (D6, `PATTERN-run-records`) writes through `friction.EnsureDir` and `friction.NotePath(frictionDir, "pr-review-send-back-<n>")`, so a later send-back or the agent's own note never overwrites it.
The note says Code-Review approved the diff these findings were raised against, gives the count by severity, each finding's summary, and the stored file's path.
An empty friction directory, Tier 2 off, writes none.

Bound: the verb lets the agent skip a send-back only where the decision record accepts it.
On the edge's fallback cases, fix-here caps nothing, as in a recipe with no Code-Review segment.
Where the edge can fire, at most `pr_review_small_max` findings are fixed in place between two send-backs; shed's reroute cap bounds the send-backs.
The verb reads no answer of the parent, so a set the agent transcribes short goes unchecked, the accepted risk of D4.

`TestPRFindingsVerb`, tier0, in a new `prfindings_test.go` over temporary record, scratch and friction directories and a fake `edge` and dirty seam, rows:
one BLOCKING finding sends back, copying `pr-findings-1.md`, marking the record and writing `pr-review-send-back-1.md`;
more than `smallMax` findings, none BLOCKING, send back;
a set within `smallMax` sends back once the record's `FixHereCount` lifts the total past it;
a set within `smallMax` answers fix-here and adds to the count;
the edge answering false, for each of its three reasons from `sendBackEdge`, answers fix-here on a BLOCKING set;
a second send-back writes `pr-findings-2.md` and `pr-review-send-back-2.md` and leaves the first note in place;
the first note's text holds the findings' count by severity, each finding's summary and the stored file's path;
an empty friction directory sends back and writes no note;
a dirty worktree, an unparsable file, a run with no record, and a failing `smallMax` or `edge` read are each refused with the messages above and change nothing;
a record already marked answers the send-back again and writes nothing;
a failing `commit` after a fix-here answers its message, and the re-run with a working `commit` leaves `FixHereCount` counting the set once.
`sendBackEdge`'s three reasons are reached through rows built over `shedengine.ProducerDef` values: no `OnReroute`, a target `EntryGate` answering false, a target `MaxBounces` of 0.

`TestPRReviewSmallMax_ReadsThePairsLayers`, tier0, built as `TestPullRequestDeps_ParentReviewWantedReadsThePairsLayers` is, writes `pr_review_small_max: 1` into the pair's local layer over a hub value of 3 and asserts `prReviewSmallMax` answers 1, and 3 with no local value (criterion D5, `run_settings` overrides it).

`Command` registers the verb; `doc.go` and `docs/overview.md`'s PR-Review section describe it.

**ImpactSummary:** One new loom verb registered in Command, writing the pull-request record, a scratch copy of the findings and a friction note; no existing verb changes.

**Commit:** `8: loom-pr-findings-verb`
