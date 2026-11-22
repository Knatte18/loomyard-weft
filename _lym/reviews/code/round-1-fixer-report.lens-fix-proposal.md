```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's F1 proposal, applied in `verbs_test.go`, through the fix-proposal lens.

No findings.

- **`textListingRows` on short or empty output:** `strings.Split("", "\n")` returns `[""]`, so `lines[0]` and `lines[1:]` never panic. An empty listing yields an empty `firstLine` and no handles, so the assertion fails with a readable message instead.
- **Handle-based row check:** the handles `T1`, `P0` and `E1` are assigned per kind, so `T1` is the task and `E1` the note. The check still catches a swapped order or a missing root row, and padding changes cannot break it. It no longer pins the `[active]` suffix or the label column. That is layout the review asked to drop, and the label filter is still pinned by the row set `[T1]`.
- **`find` check:** `HasPrefix("E1 ")` plus `Contains("later")` holds under any column width, as long as the handle is followed by whitespace. The original check had the same dependence.
- **`stepListShowsASketch`:** scanning for any line starting `S1 ` and containing `sketched` still fails when the sketch is not listed or gets another handle. It drops the sketch's position under the `sketches:` line, which the review asked to drop. A sketch listed under the wrong heading would now pass, but that is layout of the human-read output.
- **Lines left unchanged:** the sort, refusal and P0-ordering checks keep their `HasPrefix("P0 ")` and line-count assertions, so nothing the review excluded was touched.

Tool calls made: none; my context already held every changed hunk.
