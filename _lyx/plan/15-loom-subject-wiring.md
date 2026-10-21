# Card 15 — loom-subject-wiring

**Edit:**
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#loomSegmentSettings`
- `internal/loomcli#wireDarnReview`
- `internal/loomcli#loomCLI.wireDarn`
- `internal/loomcli#darnDeps`
- `internal/loomcli#TestWriteCarryOver_Real_CommitsTheRecordAlone`
- `internal/loomcli#TestWire_ReviewKeysReachTheEnv`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli/doc.go`
- `contracts/recipes/loom-recipe.yaml`
- `contracts/recipes/darn-recipe.yaml`

**Create:**
- `plan:internal/loomcli#harnessOwnFiles` -> `var harnessOwnFiles = []string{".scratch_task.json"}`
- `plan:internal/loomcli#documentSubject` -> `func documentSubject(location *lyxcwd.Location, paths []string) (shedadapters.Subject, bool, error)`
- `plan:internal/loomcli#codeSubject` -> `func codeSubject(location *lyxcwd.Location, runID string) (shedadapters.Subject, bool, error)`
- `plan:internal/loomcli#mergeSubject` -> `func mergeSubject(location *lyxcwd.Location) (shedadapters.Subject, bool, error)`
- `plan:internal/loomcli#darnRejectReview` -> `func darnRejectReview(location *lyxcwd.Location) (string, time.Time, bool, error)`
- `plan:internal/loomcli#TestDocumentSubject` -> `func TestDocumentSubject(t *testing.T)`
- `plan:internal/loomcli#TestCodeSubject_Real` -> `func TestCodeSubject_Real(t *testing.T)`
- `plan:internal/loomcli#TestMergeSubject` -> `func TestMergeSubject(t *testing.T)`
- `plan:internal/loomcli#TestDarnRejectReview` -> `func TestDarnRejectReview(t *testing.T)`

**Uses:**
- `internal/discussionparser#IdentityText`
- `internal/planparser#IdentityText`
- `internal/fabricengine#Fabric.ReadTaskPatch`
- `internal/fabricengine#Fabric.MergeInOnly`
- `internal/fabricengine#Fabric.WeftHeadSHA`
- `internal/loomshed#LatestRejectReview`
- `internal/loomcli#pairMergeBase`
- `internal/loomcli#readRecordedParentBranch`
- `internal/loomcli#writeCarryOver`

**Intent:** Loom and darn tell each Bouncer row its subject (D5), the allowance (D7) and the reject file (D6), and the recipes name the subject per row.

`documentSubject` hashes each artifact path in order, the decision record through `discussionparser.IdentityText`, so every carry-over block and every `Added after Discussion` entry never change it; an absent path hashes as absent; `Base` is the weft HEAD.
An artifact path that is a directory, as Plan-Bouncer's `_lyx/plan` is, hashes the plan the Plan-Review rubric reviews: `00-overview.md` through `planparser.IdentityText`, so Approve's flag never changes it, and every top-level `NN-<slug>.md` card file in name order; subdirectories (`archive-*/` generations), `lens-review.md` and every other file are left out.
`codeSubject` reads the recorded parent branch; a run with none (standalone) answers false, so its code segments write no records.
Otherwise it reads the merge base through `pairMergeBase` and `ReadTaskPatch` with the exclusion set: the run's own state directories (webster's run directory, the plan directory and the scratch directory, as `websterengine.UncommittedPaths` excludes them), each as a `/`-ended entry, plus `harnessOwnFiles`, the exact worktree-relative files the harness itself writes, exempted only while untracked.
The run's own records (status commits, review run directories, the settle record, carry-over) live on the weft side, so no warp diff holds them.
`mergeSubject` hashes the merge record's merged tip and merge commit, and answers false with no merge record.
Bound: a change confined to `harnessOwnFiles` reads as an unchanged subject; the verify tiers at Publish, darn's Merge-In and loom's Merge-Review bound it.

`loomCLI.wire` fills `Env.DocumentSubject`, `CodeSubject`, `MergeSubject`, `MergeInOnly` (over the recorded parent branch's tip), `WeftHead` and `RejectReview` (`loomshed.LatestRejectReview` over the rework directory); `loomCLI.wireDarn` replaces `RejectReview` with `darnRejectReview`, which reads the reject file `darnDeps`' new `RecordRejection` writes into the landing directory, whose Commit seam commits it, holding the findings and `rejected_at`.
`loomSegmentSettings` and `wireDarnReview` set `AmendMaxBounces` from `review_amend_max_bounces`.
The recipes name the subject per Bouncer row: `Discussion-Bouncer` and `Plan-Bouncer` `document`, `Webster-Bouncer` `code`, `Merge-Bouncer` `merge`, darn's `Darn-Bouncer` `code`.
The CarryOver, Approve and Commit seams must be idempotent for case 1: `ApprovePlan` already is, and a re-run of carry-over over an unchanged entry commits nothing.

`TestDocumentSubject` covers what `documentSubject` adds over card 9's `TestIdentityText`, whose rows already pin each stripped part: a decision record with a carry-over block gives the subject of the record without it (the strip is applied); a plan directory before and after `SetApproved`, and with an `archive-1/` subdirectory and a `lens-review.md` added, gives equal subjects; an edited card file changes it; an absent artifact path gives a subject and no error, and creating the file changes it.
`TestCodeSubject_Real` (integration, `hubforge` pair): a status commit and a carry-over commit on the weft side between two reads leave the subject unchanged; an untracked file outside the exclusion set changes it; an untracked `.scratch_task.json` does not; an untracked file under the run's scratch directory, a `/`-ended exclusion, does not; a standalone location answers false.
`TestMergeSubject`: no merge record answers false; a record gives a subject, and a record with another merged tip changes it.
`TestDarnRejectReview`: the file `darnDeps`' `RecordRejection` writes is read back with its path and `rejected_at`; no file reads as not found; an unparseable time is an error.
`TestWriteCarryOver_Real_CommitsTheRecordAlone` adds a second identical call that makes no commit.
`TestWire_ReviewKeysReachTheEnv`'s `wantSegments`, for the four loom segments, and `TestWire_DarnRecipe`'s `wantSegment`, for `Darn-Review`, compare each `SegmentSettings` whole, so each gains `AmendMaxBounces: 1`, the template default.
`TestWire_ReviewKeysReachTheEnv` also asserts `c.env.RejectReview` is non-nil after `wire()` of a loom seed, and `TestWire_DarnRecipe`'s darn-seed subtest also asserts `c.env.RejectReview` and `c.env.Darn.RecordRejection` are non-nil; both are direct field checks, since `envkit.NilSeams` skips these two paths through `nilLegal` and so cannot report them.

**ImpactSummary:** loom and darn wiring tells new Env seams and settings, and both embedded recipes now name a subject on every Bouncer row, which every run and recipe build reads.

**Commit:** 15: loom: tell every review segment its subject identity, the amendment allowance and the reject file
