```yaml
lens: code-pattern-fit
outcome: reported
```

**Lens code-pattern-fit: rules checked against the amended merge**

| Rule | Result |
|---|---|
| `PATTERN.md` holds 30 entries | Held. |
| `pattern/` holds 28 files; 2 entries have no background file | Held. `github-auth` and `shell-guard-table` are the two without one. |
| Orphan `pattern/` files | Held. No orphan: every file is named by a live entry. |
| Declared `PATTERN changes` landed with their declared text | Held for the entries I sampled. The 30 live IDs match the changed entries. All 70 declared removals are absent from `PATTERN.md`. |
| Parent changes added no `PATTERN-` ID or entry | Held. The parent's diff touches only `gatecli`, `gateslot`, `landingshed` and `docs/overview.md`. The parent's added lines name no `PATTERN-` ID, and `PATTERN.md` is unchanged since the settled tip. |
| Removed-ID sweep, including the parent's new gate text and the `docs/overview.md` merge | Held. The only citations of removed IDs are under `docs/research/`, which the record exempts. The other non-pattern hits are test placeholders or unrelated names (`PATTERN-x`, `PATTERN-a`, `PATTERN-fit`, `PATTERN-entry-id`). |
| Overview regrouping and gate line survived the merge | Held. The gate line carries the parent's "stops no later tier" wording, and the task's repointed references (`cwd-resolution`, `test-isolation`, `shed-driver`, `pair-teardown`) are intact. |
| `internal/landingshed/doc.go` merge | Held. The task's added line and the parent's "first failing tier" edit coexist at lines 7 and 123 with no conflict. Each stays on its own line. |
| Commit-message `Pattern-Departure:` trailers | Held. No commit in the range carries one, and none is needed. |
| Comment convention: one sentence per line, no references to other symbols | Held. The `recipe-owns-failures` sentence stands alone and names no other symbol. |

**Finding**

- **LOW, design, `internal/landingshed/doc.go:7`.**
  - The moved `recipe-owns-failures` section is one bare sentence.
  - The record says each moved rule is "condensed to the rule, its bounds and its enforcement".
  - The sentence gives only the rule. It states no bounds (Merge-In and Finalize only, merge or test failures only) and no enforcement.
  - The original entry read "A merge or test failure at Merge-In or Finalize: its fixer, a reroute or a halt to the orch."
  - The new sentence is close to that, but it sits among the Finalize paragraphs with no marker that it is the moved rule.
  - The sibling sections I looked at, such as `shedverbs`, state the behavior and its guarantee.
  - Evidence: `internal/landingshed/doc.go:7`, and `PATTERN.md` at `db7eee9bf^`, line 60.
  - Weak evidence: I found no background file for this entry, so I could not read the original enforcement.

I found no BLOCKING or MEDIUM pattern-fit issue.

**Tool calls**
1. Listed live `PATTERN-` IDs, `pattern/` files and orphans, and scanned all citations. The manifest had no cross-check of IDs against files.
2. Re-ran the scan with a corrected exclusion filter and compared against the parent's `PATTERN.md`. The first filter had not excluded `docs/research/`.
3. Matched citations against the decision record's 70 removed IDs and inspected the gate entries. The manifest lacked the removed-ID list in machine form.
4. Read the history of `recipe-owns-failures` and a sibling moved section in `internal/shedverbs/doc.go`. This was to judge the section's bounds and enforcement against the original.
