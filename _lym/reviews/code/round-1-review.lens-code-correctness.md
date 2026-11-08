```yaml
lens: code-correctness
outcome: reported
```

Task: lens code-correctness, running each changed function and branch on a concrete input.

**Per-function list**

- `describe`, draft branch, input: `draft=true`, ready record absent.
  - It calls `DraftDescription` and returns the path with `drafted:true`.
  - A failure goes through `describeFailure`.
  - The ready, run-state and owner refusals run before the branch, so they cover the draft too.
  - Correct, apart from F1.
- `describe`, record branch, no landing record: `found=false` gives the `pr open` way forward. Correct.
- `describe`, record branch, `record.PullRequest != nil`:
  - `prUpdate` re-runs `judgePullRequestVerb`, which is redundant but harmless.
  - It then pushes the branch, edits the PR from the file on disk, stamps `PushedHead`, and the verb copies `number`, `url`, `head` and `way_forward`.
  - Correct.
- `describe`, `prUpdate` fails (GitHub down):
  - The description is already committed and `DescribedAt` stamped.
  - The error says "recorded, but its pull request is not updated" and wraps the update's own way forward.
  - No inconsistent state, and a re-run is idempotent.
  - Correct.
- `describe`, `Landing` read fails after record: the error says "recorded" and names `pr update`. Correct.
- `describeFailure`: an error that already has "way forward:" is kept, any other gets one. Correct.
- `RecordBeamDescription`, file missing: `ValidateDescription` returns exactly one `CheckMissing` finding, so the missing-file refusal names `--draft` and the commit closure is not called. Correct.
- `RecordBeamDescription`, empty file or no heading:
  - The `CheckEmpty` and `CheckNoHeading` findings go to the generic finding branch.
  - The refusal names the file and the verb, and nothing is committed or stamped.
  - Correct.
- `RecordBeamDescription`, valid text: it calls `commit()`, so the stamp is written first and then the commit.
  - If the commit fails, `DescribedAt` is stamped but nothing is committed.
  - The old code ordered it the same way and a re-run is idempotent, so I judge it not a regression.
- `RecordDescription` closure in `deps.go`: `beamOf` replaces `beamAndParent`, so the origin is no longer required. Correct, since no commit list is read.
- `DraftBeamDescription`, empty brief and no commits: it writes `# Title\n`, which fails validation at record time, the intended prompt for the orch to edit. Empty title writes `# `, which is rejected at record time. Correct.
- `DraftBeamDescription`, existing file: replaced whole without a check. See F1.
- `requireBeamDraftDeps`: names the first unset field. Correct.
- `judgeDescribed` (land): the validation message now points at `summary.md`. Correct.
- `prUpdate` reads the description from disk, not from the commit. See F1.
- `dobbyengine` notice text: a string change only. Correct.

**Findings**

- **F1, MEDIUM, design**, `internal/beamcli/describe.go` (the draft branch) with `internal/landingshed/beamdescribe.go` (`DraftBeamDescription`, the `os.WriteFile` call).
  - `--draft` overwrites `summary.md` with no guard against the orch's own text.
  - Running `--draft` after the orch has edited or recorded a description destroys that text.
  - It also leaves an uncommitted draft on disk that `pr update`, `pr open` and `land` read from disk.
  - The decision record says the orch and the operator may override at any time, so a verb that silently discards their text contradicts that.
  - Fix: have the draft refuse, naming the file, when `summary.md` already exists, or write the draft to a different path.
  - Confidence: moderate; the guard is a design choice.

No other defects found in this lens.

**Tool calls:** none. My context already held everything this lens needed.
