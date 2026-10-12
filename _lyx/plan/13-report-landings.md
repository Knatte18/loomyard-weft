# Card 13 — report-landings

**Create:**
- `internal/reportengine#Landing`
- `//internal/reportengine/landings_test.go`

**Edit:**
- `//internal/reportengine/doc.go`
- `internal/reportengine#fakeRepo`
- `internal/reportengine#fakeRepo.BlameLines`
- `internal/reportengine#allowedImports`
- `//cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/fabricengine#TaskCommitOf`
- `internal/loomengine#LandingCheckpointSlug`
- `internal/boardengine#ParseBoardTasks`
- `internal/boardengine#BoardFileRel`
- `internal/gitrepo#Repo.PreImageLines`
- `internal/gitrepo#Repo.BlameLines`

**Intent:**
The report gains landings by kind (D10) and the fresh-code fix share (D11), in a new file `landings.go`; `Report.Landings` lists every landing in range, and a full run's `RunRow.Landing` points at its own.

Landings (D10):
- The landings are the first-parent commits of `ParentTip` committed at or after `Since`, from one `Code.Log(ParentTip, LogQuery{FirstParent: true, Since: Since minus FreshDays days})`; the older commits that walk returns only date blame owners.
- Exact join, applied to every tag the selection read before any time join: the task commit is `fabricengine.TaskCommitOf` of the newest commit carrying one in `Records.Log(tag commit, LogQuery{FirstParent: true})`, walked in growing pages (`Limit` 64, then doubling, re-reading from the tag) until a commit carrying a trailer is found or the walk returns fewer commits than its `Limit`.
Loom's per-transition records commits carry no trailer, so that commit often sits several commits below the tag (in the hub's own archives, webster's last record-batch commit); a tag whose history carries none gets no exact join.
  The landing is the first-parent commit that is the task commit (a fast-forward), has it as a parent (a merge), or has its tree (a squash).
  For a fast-forward, the first-parent commits after the nearest older landing and before the task commit are the task branch's own: they belong to that landing and are neither landings nor `unmapped`.
- Time join, for a tagged run the exact join left unmapped and for a slug with checkpoints after its newest tag (a pair not yet torn down): the checkpoints are the records commits whose subject `loomengine.LandingCheckpointSlug` names the slug, from card 10's records walk.
  A tagged run's window opens at the newest checkpoint committed after the slug's previous tag's commit and at or before this tag's commit; an untagged slug's opens at the newest checkpoint after its newest tag's commit, and a slug with no archive tag opens at its newest checkpoint.
  The window closes at the first board commit after the checkpoint whose `board.json` (`ParseBoardTasks` over `BoardFileRel()` at that commit, from one `Board.Log(BoardTip, LogQuery{})`) shows the slug's entry with status `done`.
  The landing is the first first-parent commit committed inside the window that no exact join claimed; every comparison reads committer dates.
  No checkpoint in the interval, or no done mark after it, means no time join.
- Within a source, two runs claiming one commit both become unmapped and the clash is a finding; a landing neither source maps is kind `unmapped`.
- A tag mapped by the exact join whose loom history at the tag holds no `Finalize` `done` entry was landed by hand, and its landing has `Hand` set; a time-joined landing is never `hand`.
  For a full run, `RunRow.LLMSteps["landing"]` is 1 for a hand landing and 0 otherwise (D8); a partial tag (D4: no `Finalize` entry at all) has no `RunRow`, so its hand landing shows in `Report.Landings` with its tag and in the week's `Hand` count alone.
- Otherwise the kind comes from the slug's board labels as recorded in the newest board commit at or before the landing that holds the entry, so an entry pruned since still reads: `fix` when they include `bug`, `feature` for `feature` or `enhancement`, else `other`; an entry no board commit holds is `other` with a finding naming the slug.
- A tagged full run whose `Finalize` is done and whose batten record's history never reaches `Land` `done` keeps its label kind and has `LandIncomplete` true; with the batten source absent, and for an untagged slug, the flag is absent.
- An untagged slug's landing appears in `Report.Landings` only, never in `Runs`.

Fresh-code fixes (D11), for each `fix` and `unmapped` landing in range and no other:
- `Code.PreImageLines(landing)` gives the lines it modifies or deletes; paths ending `_test.go`, paths under `docs/` and paths ending `.md` are dropped.
- With no line left the landing is `no-pre-image`; otherwise each file's lines are blamed with `Code.BlameLines(first parent, path, lines)`, and each line's owner is the landing that brought the blamed first-parent commit: itself, or the fast-forward landing its task-branch commit belongs to.
- A line is fresh when its owner was committed within `FreshDays` days before the landing; the landing is `fresh` when more than half its lines are, `aged` otherwise, an owner the walk did not reach counting as aged.
- A fast-forward's own task-branch commits are never blamed, and a `feature` or `other` landing is never blamed.
`Landing` holds `Commit`, `Committed`, `Week`, `Kind`, `Slug`, `Tag`, `Join` (`exact` or `time`, empty for unmapped), `Hand`, `LandIncomplete` (`*bool`) and `Fresh` (`fresh`, `aged`, `no-pre-image`, or empty when not blamed).

`seam_enforcement_test.go`'s `allowedImports` gains `boardengine`.
`fake_test.go`'s `fakeRepo` gains a call log for `BlameLines` and builders for a parent-branch history, a records commit with a trailer, a checkpoint commit and a board commit.
`doc.go` describes both joins, the kinds and the fresh-code rule.

`TestCompute_Landings` (new, in `landings_test.go`) is table-driven, each row asserting the exact `Report.Landings`, the affected runs' `Landing`, LLM steps and findings, and the `BlameLines` call log:
- a fix whose modified lines mostly came from a landing two days earlier is `fresh`, ten days earlier `aged`, exactly half from a recent landing `aged`, a pure-addition fix `no-pre-image`, and a fix touching only a `_test.go` file, a `docs/` file and a `.md` file `no-pre-image`;
- a full run mapped by the exact join whose loom history at its tag holds a `Finalize` entry but no `Finalize` `done` is a `hand` landing with one landing LLM step, and a partial tag with no `Finalize` entry mapped by the exact join is a `hand` landing carrying its tag in `Report.Landings` and adds no run;
- a landing whose pair is not yet torn down, on a slug with no archive tag, maps through its checkpoint and board done commit, keeps its label kind, is not `hand`, has no `LandIncomplete` and adds no run;
- a commit mapping to no run is `unmapped`, is blamed, and its share never mixes into a fix's;
- a slug with two runs, each with its own checkpoint and board done commit, maps each landing to its own run, and a run with two checkpoints in its interval opens its window at the newer one;
- a run with no checkpoint in its interval, and a run whose board entry was never marked done after its checkpoint, get no time join;
- a fast-forward landing of a task branch with three commits is one landing and no `unmapped` commit, and the call log holds no blame of its task-branch commits;
- a run whose `Finalize` landed but whose batten `Land` never reached `done` keeps its label kind and has `LandIncomplete` true, not `hand`;
- a run with no checkpoint commit maps through its task commit's trailer by commit (fast-forward), by parent (merge) and by tree (squash), one row each, the merge row's trailer sitting three trailer-free commits below the tag;
- a commit in a run's window that another tagged run's exact join claimed is not taken by the time join, and two runs whose exact joins claim one commit both become unmapped with a finding;
- labels `bug`, `enhancement` and `chore` give `fix`, `feature` and `other`, read from the last board commit before the landing even when a later board commit pruned the entry;
- an exact-joined landing whose slug no board commit holds is kind `other` with one finding naming the slug, and is not blamed;
- a `feature` landing and an `other` landing leave the call log empty;
- a landing committed before `Since` is left out.

`cmd/lyx/testdata/test-budget.yaml` raises `internal/reportengine` to 6.

**ImpactSummary:** Extends the unreleased report engine and its test support; nothing outside `internal/reportengine` and the budget file changes.

**Commit:** `13: report engine: landings and the fresh-code fix share`
