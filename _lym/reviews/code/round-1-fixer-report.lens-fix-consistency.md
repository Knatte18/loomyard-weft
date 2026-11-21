```yaml
lens: fix-consistency
outcome: reported
```

Task: check the change set through the fix-consistency lens, looking for twins left stale.

Findings:

1. **MEDIUM** — `contracts/stencils/burler/burler-assist-cluster-fan.md:29`, the twin of the `burler-cluster-fan.md` edit made for F6.
   - Its consolidation sentence still reads "tag every kept finding's frontmatter with an `origin:` key (`lens:<name>` or `handler`)", with no `origin: webster` exception.
   - A round whose cluster fan comes from an assistant reviewer uses this stencil, so the halt finding loses its `origin: webster` there. The changed-file rule then applies to it, which is the defect F6 describes.
   - Evidence: grep line 29 of that file.

2. **MEDIUM** — `contracts/stencils/burler/burler-parent-consolidation.md:15` and `:22`, a second consolidation rule that sets an origin.
   - A kept parent finding is tagged `origin: {{.parent_origin}}`, and a merge keeps that origin.
   - The F6 exception was added to the cluster-fan and assist-consolidation stencils only. A halt finding that duplicates a parent finding, or that the parent raises, gets `origin: parent` with no rule saying webster wins.
   - The exception is also missing from this file's rules.
   - It is a gap in the fix's coverage of F6, not a contradiction.

3. **LOW** — `internal/websterengine/doc.go:~495` still says a create-not-done or rename-not-done finding "offers three ways forward", the second being a parent decision recorded with `--drops-create`.
   - After F3, a rename-not-done finding offers only two: the amendment and recover-batch.
   - Evidence: `createTargetWayForward` in `recordbatch.go` returns the amend and recover-batch text when `droppableCreate` is false. The `renameWays` test row pins that.
   - This doc sentence is the twin of the `failCardNotDone` doc comment that F3 did update.

No stale twin found for F7, F8, F14, F15, F16 or F1/F2.
- The `--drops-create` wording agrees across `loomcli/doc.go:779`, `discussionparser/doc.go:19` and `contracts/specs/loom-discussion-spec.md:56`.
- No other reference to `planReviewRunSubdir` exists.

Tool calls:
1. Bash grep for `record each folded call again`, `planReviewRunSubdir`, `names a loom verb`, `drops-create` and `Drops Create` across `.go`, `.md` and `.yaml`, to find twins of the changed text. My context held none of those files.
2. Bash grep for origin and halt-reason text across `contracts/stencils/burler`, and for tests that pin the parent-directive wording. My context held only two of the burler stencils.
3. Bash grep for `origin` in `burler-assist-cluster-fan.md` and `burler-parent-consolidation.md`, plus `websterengine/doc.go` lines 492–500 and a grep in `loomcli/doc.go`. I had not read those files.
