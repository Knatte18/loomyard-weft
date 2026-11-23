```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens. I stepped through each function and branch the diff changes and checked the result.

**Function and branch list**

- **`rememberRuns`, run ahead of Land, reading on (`requests = {task: true}`)**: the memory becomes Requested=true, Read=true and is stored. Correct.
- **`rememberRuns`, run ahead of Land, lookup fails (`requests` empty)**: the earlier reading is kept and the memory is stored unchanged. Correct.
- **`rememberRuns`, past Land, Requested=false, follow-up set (the beam has a recorded pull request; the member row whose land_notice is off)**: Read is true, so the notice is decided. With Requested false, Line stays empty, nothing is owed and the memory is deleted. No line goes out. This matches decision record §1. Correct.
- **`rememberRuns`, past Land, Requested=true, follow-up set**: Line is `landNoticeLine(run, followUp)`, so the follow-up stays on the landed line. The line is owed and the memory is stored. Correct.
- **`rememberRuns`, past Land, Requested=true, follow-up empty (no pull request, the prime's dobby, or a nil seam)**: the plain landed line goes out. Correct.
- **`rememberRuns` with decides=false (the landing record read failed)**: the memory is stored and nothing is decided until a later tick reads the record. Correct, and this branch is unchanged.
- **`rememberRuns`, past Land, never read**: the run stays undecided. This branch is unchanged. Correct.
- **`rememberRuns`, empty `runs`, nil `requests`**: there is no loop work, and memories that owe nothing are pruned. Correct.
- **`landNotices` after the board read was removed**: it no longer calls `deps.Tasks`. Before, a failed board read in this step was only logged and swallowed, so the step has no error path left to lose. The tick's own start step still reads the board at `internal/dobbyengine/starts.go:100`, so a board failure still surfaces there. Correct.
- **An older state record carrying `"member": true`**: `LandNoticeState` no longer has the field, so `encoding/json` ignores the key and the record loads. On the next rewrite through `UpdateState` the key is dropped. Nothing reads it, so no state is lost that this binary acts on. If such a run was ahead of Land with Requested=false, it now gets no line, which is the intended new rule. Correct.
- **An older record with an already decided `Line` for a member**: it is still owed and posted by `settleLandNotices`. Correct; the line was decided under the old rule, and resending it is harmless.
- **`pullRequestFollowUp`, `daemon` struct (`landNoticeBoard` removed), `Deps.PullRequestRecorded` doc**: no logic changed. Nothing references `landNoticeBoard` any more (grep in the manifest). Correct.

**Findings:** none.

**Tool calls**
- `sed -n 1,200p internal/dobbyengine/landnotice.go` plus a grep for `deps.Tasks`: I needed the whole post-diff `rememberRuns`, `landNotices` and `unreadLandNoticeRuns` to step through, and the board read's other caller. My context held only the diff hunks.
