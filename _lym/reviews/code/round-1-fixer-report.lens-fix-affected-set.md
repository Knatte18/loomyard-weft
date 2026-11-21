```yaml
lens: fix-affected-set
outcome: reported
```

Searches for F8, F6, F11 and F7 are done; F1, F3, F4, F5, F9, F10, F12, F14, F15 and F16 were left unchecked. Two findings below.

**MEDIUM: F6 fix missed the assistant-fanned twin stencil.**
- Changed text: `contracts/stencils/burler/burler-cluster-fan.md:32` and `burler-assist-consolidation.md:11` now keep `origin: webster` on the halt finding.
- Not changed: `contracts/stencils/burler/burler-assist-cluster-fan.md:29` is the assistant's fanned part. It still says "tag every kept finding's frontmatter with an `origin:` key (`lens:<name>` or `handler`)".
- Effect: `clusterRulesBlock` (`internal/burlerengine/prompt.go`) selects `assistClusterFanStencil` for an assistant round, so that round still strips `origin: webster` from the halt finding.
- Evidence: grep of `origin:` across `contracts/stencils/burler/*.md` lists that line.

**LOW: F6 left a perishable rule in two places.** `burler-review-format.md:55` ("In a cluster round, each finding also carries an `origin:` key — `lens:<name>` ... or `handler`") is the format the finding cites. It does not name the `webster` exception. Review the line against the fix.

**Checked, no gap:**
- **F8:** only one stencil holds the "record each folded call again" bullet, `parent-directive-parent.md:33`.
- **F11:** the `PostBatchInputs{` sites in `weaverengine/record.go:162` and `resume.go:440`, and `postBatchInputs` in `websterengine/recordbatch.go:896`, are the only constructions. `resume.go:448` sets `DecisionRecordPath`. The `record.go:170` site is outside the grep output and I did not read it. Only the run path has a weaver test; `resume.go` has none, and the review did not ask for one.
- **F7:** no other "names a loom verb" statement exists in `internal` or `contracts`.

Tool calls:
1. `grep` for the F8 bullet text across the repo, to find twin stencils.
2. `grep` for `origin:` tagging in `contracts/stencils/burler/*.md`, for the F6 twins.
3. `grep` for `PostBatchInputs{` and `DecisionRecordPath` in non-test Go, for the F11 sites.
4. `grep` for "names a loom verb" in Go and Markdown, for the F7 twins.
5. `sed` and `cut` on `burler-assist-cluster-fan.md:29` and `burler-parent-consolidation.md:15-16`, to read the full tag sentences.
