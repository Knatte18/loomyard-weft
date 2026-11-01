```yaml
lens: code-correctness
outcome: reported
```

**Lens code-correctness: case run per function or branch**

Each case is judged from the diff and the source I read, not by running the code.

- `Parse` and `ParseData` (`internal/summaryparser/summary.go`):
  - Case: an unreadable file, an empty body, no heading, and an empty title.
  - Correct: `Parse` returns the same read error and delegates the rest. Error text now uses `source`, and the sentinels are unchanged. `websterengine` and `landingshed` still call `Parse` unchanged.
- `numstatLines` (`landed.go`):
  - Case: empty output, a binary row `-\t-\tf`, a rename row, and a tab inside a filename.
  - Correct: it returns 0, skips the binary counts, counts the rename row, and `SplitN(…,3)` keeps the tab in the name.
- `codeRepo.ChangedLines` (`landed.go`):
  - Case: a landed commit with no parent.
  - Wrong: see F2.
  - Case: a merge commit.
  - Correct: `sha^1` diffs against the first parent.
- `findLanding` (`landed.go`):
  - Case: the description is absent at the tag.
  - Correct: `Landing{Missing: noDescription}`.
  - Case: a malformed description.
  - Wrong in effect: see F1.
  - Case: a commit exactly at `span.To + 1m`.
  - Correct: the bound is inclusive.
  - Case: several commits share the title.
  - Correct: the newest within the slack-widened span wins.
  - Case: the match against the commit subject.
  - Correct, because the landing passes the summary title as the merge title (`finalize_pr.go:223`). I did not read `CommitMessage`, so the subject-equals-title assumption rests on that.
- `finishedRun` (`shed.go`):
  - Case: darn and tapestry with a Finalize entry, a sampler run, no record, and a still-running worktree.
  - Correct: darn and tapestry count, and the other three are not finished.
  - Case: the Finalize name for darn.
  - Correct: it is the generic `loomshed.NameFinalize`.
- `useShedRecord` (`shed.go`):
  - Case: the `Entries[0]` access.
  - Correct: `readShedRecord` returns nil for an empty history.
- `recentRuns` (`count.go`):
  - Case: zero `since` with a non-zero `touched`.
  - Correct: the run passes the filter.
  - Case: `since` after `touched`.
  - Correct: the run is dropped and not listed as skipped.
  - Case: `last` of 0 or less.
  - Wrong: see F3.
- `parseSince` and `run` (`main.go`):
  - Case: an empty value, a valid date, and "yesterday".
  - Correct: zero time, local midnight, and an error.
  - Case: `-since` set together with `-last`.
  - Correct: `limit` is forced to 0.
- `writeRecipes` (`recipes.go`):
  - Case: a recipe whose runs have zero landed commits.
  - Correct: both columns show `n/a`.
  - Case: landed commits with 0 lines.
  - Correct: lines per run shows 0 and cost per line shows `n/a`, so there is no division by zero.
  - Case: runs that are never zero in the per-run divisions.
  - Correct: rows only exist for a recipe with at least one run.
  - Case: `unknown` recipe and a run with a nil `Landing`.
  - Correct: both are grouped and listed under "No landed commit".
  - Case: Unpriced costs.
  - Correct: `precise()` mirrors `String()`.
- `writeCostPerRun` and `partOf` (`segment.go`):
  - Case: segment-to-column lookup.
  - Correct: every `partOf` name is in `segmentNames()`, so `slices.Index` never returns -1.
  - Case: columns for segments no run holds.
  - Correct: those columns are omitted.
  - Case: roles `describe`, `pr-reviewer`, `fixer`, `fix-reviewer`, and their `+sub` forks.
  - Correct: they land in Landing, so a tapestry run's total is unchanged. Its describe cost moves from the Other segment to Landing.
  - Case: the darn, drift and code-review role strings.
  - Unverified: my calls found `darnRole = "darn"`, `drift-baseline`/`drift-verify`, and the burler suffix `burler-review`. I did not confirm the `code` prefix for darn's Code-Review rounds or `-bouncer-judge`. `segment_test.go` pins the names (`code-burler-review`, `code-bouncer-judge`), not the producers' constants.

**Findings**

- **F1 — MEDIUM, design — `tools/tokencount/landed.go:88-90`, caller `main.go:331-334`**
  - A single run whose archived change description does not parse (empty file or no `# ` heading) makes `findLanding` return an error.
  - `run` returns that error, so the whole report fails.
  - Every other missing-landing case (no description, no titled commit) is recorded as a `Landing.Missing` reason and the report goes on.
  - The focus asks for exactly this case to be handled "without failing the whole report".
  - `TestFindLanding` pins the failing behavior ("a change description that does not parse").
  - Fix: return `Landing{Missing: "change description does not parse"}` and keep the error text in the reason.

- **F2 — LOW, design — `tools/tokencount/landed.go:44-46,257`, caller `main.go:331`**
  - If the title-matched commit has no first parent, such as a root commit, `git diff sha^1 sha` fails.
  - `findLanding` wraps the failure and the whole report aborts. A fabric-style `Missing` outcome would be more consistent.
  - Fix: treat a `sha^1` failure as a landing with no countable lines, or diff against the empty tree.

- **F3 — LOW, design — `tools/tokencount/count.go` (`recentRuns` loop), `main.go:282`**
  - `last <= 0` now means unlimited. Before the diff, `-last 0` counted nothing and ended in the "no runs" error.
  - With no `-since`, `-last 0` or a negative `-last` now counts every finished run, which is an undocumented change in flag behavior.
  - Fix: pass `limit = 0` only when `-since` is set, and refuse a non-positive `-last` otherwise, or document the new meaning.

**Tool calls**
1. Read `CommitMessage` in `summary.go`, the Finalize and role-name constants, and the burler role strings. My context did not hold them; `CommitMessage` printed nothing.
2. Grepped `landingshed` and `githubclient` for the squash and merge title, to confirm the landed subject is the summary title; my context did not show it.
