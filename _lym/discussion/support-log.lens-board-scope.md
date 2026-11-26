```yaml
lens: board-scope
outcome: reported
```

Lens: board-scope. I checked the record's fit with the board entry, the touching entries, the peers and the parent commits.

**Findings**

1. LOW, D4/D5 (the index row). The entry asks for "a short index row per landed task, so `lym report` can read them without walking directories". The record reads that as one file per run, inside each archive tag, rather than one shared index. This is an auto-pick (Q1) with its rationale in Rejected alternatives. The board does not rule it out, so it is not a contradiction. Evidence: the entry body's Idea list, and the support log's Q1.

2. LOW, Goal/Scope (Describe text "as it was merged"). The entry asks for the Describe text as it was merged. The record keeps `_lym/landing/summary.md` and the landing commit in the index row. It never says that the landing commit's message is the merged text when a pull request's body was edited on GitHub. Issue #228 in `describe-review-and-sync` shows the PR body can drift from the file. That entry fixes the drift, but its parent task `pr-modes-no-agent` has not started. A line naming the landing commit's message as the as-merged text would close this goal point. Evidence: the entry body's "The Describe text as it was merged", and #228's text in the `describe-review-and-sync` body.

**Goal points of the entry**
- Decision record with its `Added after Discussion` entries at landing: met. It is already tracked and in the archive tag, as the Goal and Scope "Out" state.
- Code-Review findings and fixer reports: met. They are already tracked; D3 adds the landing fixers.
- The parent's rulings and answers: met. D2 covers them, plus the `Added after Discussion` entries.
- PR-Review findings: met (D1).
- Describe text as merged: met in part; see finding 2.
- Drift verdict: met. The drift records are tracked, and the verdict goes into the D4 row.
- Index row per landed task, read by `lym report`: met as interpreted (D4, D5); see finding 1.
- Read first what survives today and how landing archives: met. The Goal and support log Turn 1 record it.
- Decide the weft path and the retention: met. The path is the run's records plus the archive tags, and D6 keeps them forever.
- Findings no longer only in a scratchpad: met (D1 to D3).

**Entries, peers and parent commits checked**
- `pr-modes-no-agent`: no conflict. It has not started, and the Open risks name that it removes D1 and D2's PR-Review paths.
- `describe-review-and-sync`: no conflict; the Open risks note its new reviewer artifact.
- `scope-and-earlier-checks`: none.
- `dobby-run-watch`: none.
- `hardener-works-study` (note): none. It reads `_lym/reviews/harden/` from the records, which matches the record.
- Peer `config-keys-nested`: none. It renames the `pr_review_*` config keys only, and D1 to D6 add no config key.
- Peers `sandbox-closed-area-build`, `reed-client-and-bus-guard`, `creel-refusal-after-cd`: none, since their changed paths don't overlap.
- Parent commits: `lym loom peers` lists none.

**Decision notes**
- `lym board list` shows no `Decision for keep-landed-records:` note, and none was unreachable.

**Tool calls**
- `lym loom peers`: the peers and parent commits were not in my context.
- A grep of the `config-keys-nested` decision record: the content of that peer's record was not in my context.
