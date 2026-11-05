```yaml
lens: fix-consistency
outcome: reported
```

Task: check the F1 change set (the `doc.go` sentence and the added "Held by review alone" lines) for sibling disagreements and missed twins.

**Findings**

1. **LOW** — `pattern/PATTERN-shed-driver.md:106,110,114`, the twin left unchanged.
   - Summary: the cluster carries `(test)` in `PATTERN.md`, but three of its `Rule:` lines have no `(test)` and got no "Held by review alone" line.
   - This breaks the new `doc.go` sentence that "the cluster's background file marks each member rule that is review-only".
   - Evidence: a grep of shed-driver's `Rule:` lines not ending in `(test)` returns lines 106, 110 and 114: the stop rule table, the transient-failure mark and "Adding what every phase machine needs". The file has 0 matches for "Held by review alone".
   - The review's location list omitted shed-driver, but the fix's own condition (carries `(test)` and has a review-only member) covers it.
   - The nine other clusters in `PATTERN.md` that carry `(test)` and have a background file have no non-`(test)` `Rule:` line, so none is affected. `PATTERN-github-auth` has no background file.

2. **Checked, no finding** — the nine files the fixer edited:
   - Each non-`(test)` `Rule:` line in those nine files has the marker line, and the fixer's 12 insertions match the grep count of 12.
   - The marker is a plain line that the `pattern/` file-naming checks do not read.
   - `doc.go` agrees with the `(test)` definition at `doc.go:71`.
   - The gate's tier-0 `./internal/pattern` run passed, as shown in the transcript.

**Tool calls**
1. One Bash grep over the other `(test)`-carrying clusters' `Rule:` lines and the marker count. My context held only the files the fixer read, not these.
