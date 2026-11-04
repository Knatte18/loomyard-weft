```yaml
lens: fix-affected-set
outcome: reported
```

Task: check the fix's coverage of the sets behind the review's findings, through the affected-set lens.

**LOW — `doc.go:160`, the package doc paragraph on the batching nudge**
- The summary line says the hook "adds one line to the session's context when its recent calls break lym's batching rules".
- F9 reworded the same "session" scope in the `hook-batching` verb's help text and the `context_tokens` comment.
- This is the same stale wording, and it was left as it was.
- Lines 166–168 of the same paragraph already say a subagent's call is judged against its own transcript. So the summary line now disagrees with them.
- Suggested fix: change "the session's context" and "its recent calls" to the calling chain's.

**LOW — `doc.go:172`, the lookup command list**
- The command list is the same table F2 changed, and it says the git reads have "no write, substitution or redirect into a file".
- After F2, a git read with `--output`, `--ext-diff` or a pager option is also excluded, and the doc doesn't say so.
- This is wording only, because the existing phrase "no write" covers the case loosely.

**Checked, no finding**
- **F4 and F9:** the `MaxNudges` doc comment, the `max_nudges` and `context_tokens` comments in `template.yaml`, and the verb's Long text now agree.
- **`max_nudges` and `context_tokens` sites:** the grep found no other site describing either key as session-only.
  - `internal/orchcli/status.go` and `internal/orchengine/state.go` are unrelated.
  - `docs/research/.../report.md` is a dated measurement report, which the review rejected.
- **F2:** I found no summary or doc outside the Go files that names the Bounds. The only prose that mentions the git reads is `doc.go`, covered above.

**Not checked:** F1, F3, F6, F7 and F8 are tests with no cross-site set to search. The F5 table entries are all in one file, whose diff the context already shows.

**Tool calls**
1. `grep` for `max_nudges|context_tokens|MaxNudges|session's recent` across the repo, to find every site that describes the cap or the context size; the context held only the files the review named.
2. `grep` for `rg --pre|git diff|ext-diff|Bounds` across the repo, to find prose naming the lookup bounds; the context held no such search. It returned nothing relevant.
3. `Read` of `doc.go` lines 155–184, because the first grep named `doc.go` as holding the package-level description of the nudge, which the context lacked.
