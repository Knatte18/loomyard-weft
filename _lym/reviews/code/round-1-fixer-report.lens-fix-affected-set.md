```yaml
lens: fix-affected-set
outcome: reported
```

Task: rerun each finding's set search and check the fix covers every member.

**Findings**

1. **MEDIUM** — `docs/research/twill/twill-discussion-bench/deviations.md:5`: the F1 fix left one reference unchanged.
   - The fix's `replace_all` matched the backticked form `` `twill-discussion-bench.md` ``, so it fixed line 1 only. Line 5 has the name unbackticked.
   - A fresh search for `twill-discussion-bench\.md` across `docs` and `internal` returns exactly one hit: `deviations.md:5`, `see twill-discussion-bench.md "Deviations, round 4"`.
   - The review's fix named lines 1 and 5, and line 5 still points at a file that no longer exists in that folder.

**Searches with no further findings**

- F2: a search of `docs` and `internal` for `docs/research/glyph-audit`, excluding the `calls.tsv` and `findings.tsv` captures the review rejected, returns no hits.
- F3: every `data` folder under `docs/research` is at the study level (`topic/study/data`, depth 3 or 4 below `docs/research`). So `parts[4:len-1]` with `len > 5` still skips all of them. No topic-level `data` folder exists, so no real file is newly scanned.

**Tool calls**
- grep for `twill-discussion-bench\.md` over the repo and then `docs`/`internal`: reruns F1's own search, which my context lacked.
- grep for `docs/research/glyph-audit` over `docs`/`internal`: reruns F2's own search.
- `find` of `data` dirs under `docs/research` with depths: F3's set, to check no real file changes scan status.
