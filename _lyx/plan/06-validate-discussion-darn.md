# Card 6 — validate-discussion-darn

**Create:**
- `plan:internal/loomcli#discussionHistory` -> `func discussionHistory(location *lyxcwd.Location) func() ([][]byte, error)`

**Edit:**
- `internal/loomcli#loomCLI.validateDiscussionCmd`
- `internal/loomcli#loomCLI.wireLightweight`
- `internal/loomcli#loomCLI.wireDarn`
- `internal/loomcli#TestValidateDiscussionCmd`
- `internal/loomcli#TestGateParity_DiscussionGate`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWireLightweight_FillsThePathsWithoutLoadingAnyConfig`
- `contracts/specs/refusal-spec.md`
- `pattern/PATTERN-gate-self-check-parity.md`
- `docs/overview.md`

**Uses:**
- `plan:internal/fabricengine#CommittedAnchoredFileHistory`
- `plan:internal/loomshed#ValidateDarnDiscussion`
- `plan:internal/loomshed#NewDarnDiscussionGate`
- `internal/loomcli#commitDiscussion`
- `internal/loomengine#DiscussionDecisionRecordRel`

**Intent:** Make a darn writer's clean `lyx loom validate-discussion` run the darn check and commit the record and support log, so the record is committed before any code is written (D4), and wire the history seam on a darn build.

`discussionHistory`, in `internal/loomcli/wiring.go` beside `commitDiscussion`, returns the closure that reads `fabricengine.CommittedAnchoredFileHistory(location, loomengine.DiscussionDecisionRecordRel())` on each call and opens nothing when built.
`wireDarn` sets `c.env.DiscussionHistory = discussionHistory(location)`, beside the other darn seams, so the Darn-Write row's `discussion` gate resolves to the darn gate (card 5).
`wireLightweight`, which `validate-discussion` arms through and which already knows `c.recipe`, sets `c.env.DiscussionHistory` and `c.env.CommitDiscussion` (through `commitDiscussion(location)`) when `c.recipe` is `shedrun.RecipeDarn`, and leaves both nil otherwise; its doc says the darn verb reads committed history and commits through the fabric at call time, still loading no module config.

`validateDiscussionCmd` runs `loomshed.ValidateDarnDiscussion(c.env.DecisionRecordPath, c.env.SupportLogPath, c.env.DiscussionHistory)` when `c.env.DiscussionHistory` is set and `discussionparser.Validate` otherwise, mapping findings and I/O errors onto the envelope exactly as today.
On a clean darn check it calls `c.env.CommitDiscussion()` before the ok envelope; a commit error is a refusal, `loom: validate discussion: commit the discussion files: <err>; way forward: re-run "lyx loom validate-discussion"`, exit 1.
A later clean run commits again, a no-op over an unchanged record.
Its `Short` and `Long` say that on a darn run it is the Darn-Write row's gate check, which also refuses a record missing an `Added after Discussion` entry its commit history held, and that a clean run commits the two files.

`TestValidateDiscussionCmd` gains darn rows whose `build` sets the history seam and a counting commit seam on the fixture's `loomCLI`, and the row struct gains the commit count each row wants (0 for the loom rows, which set no seam): a clean record commits once and exits 0; a record missing an entry a history version holds exits 1 naming the heading and commits nothing; a commit failure exits 1 with the way forward.
`TestGateParity_DiscussionGate` gains darn cases (clean, dropped entry) and builds `loomshed.NewDarnDiscussionGate` over the fixture's seams when the case sets the history seam, comparing its verdict with the verb's.
`TestValidateDiscussionCmd`'s `//testtiming:keep` reason names the darn rows' commit count and dropped entry beside its loom cases, the header comment of `internal/loomcli/parity_test.go` names `NewDarnDiscussionGate` beside `NewDiscussionGate`, and the header comment of `internal/loomcli/validate.go` names the Darn-Write row's gate among the gates the verbs mirror.
`TestWire_DarnRecipe` asserts `c.env.DiscussionHistory` is set on a darn seed and nil on a loom seed.
`TestWireLightweight_FillsThePathsWithoutLoadingAnyConfig` asserts both seams nil for its loom-recipe `loomCLI` and set for a second one whose recipe is darn.

`contracts/specs/refusal-spec.md`'s loom section gains a row for the new refusal (`validate-discussion: commit failed on a darn run`, transient, the re-run way forward).
`pattern/PATTERN-gate-self-check-parity.md` gains the pair: the Darn-Write row's `discussion` gate and `validate-discussion` on a darn run, both through `loomshed.ValidateDarnDiscussion`.
`docs/overview.md`'s discussionparser entry names `loomshed.NewDarnDiscussionGate` beside `NewDiscussionGate`, and its `validate-discussion` line says a darn run's verb also checks the record's history and commits the files on a clean run.

**ImpactSummary:** `validate-discussion` changes behavior on a darn run only, gaining a commit and one refusal; a loom run's verb and wiring are unchanged.

**Commit:** `6: validate-discussion on a darn run checks history and commits the record`
