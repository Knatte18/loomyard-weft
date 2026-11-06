```yaml
lens: fix-affected-set
outcome: reported
```

Task: lens fix-affected-set, checking whether each fix covers every member of its finding's set. **No findings.**

Findings I left unchecked: none.

- **F1:** `pattern_clusters` is declared in one struct, `Part`, in `internal/raddleparser/proposal.go:76`. The new `Parse` tests cover the nested key at both levels: the value parse and the unknown key inside a cluster. No other type carries the key.
- **F2:** `writeRecords` (`internal/loomshed/raddlefile.go:278-301`) writes the mapped record to the project slug (when `Project` is set) and to each part slug. The fixed subtest reads exactly those records: `part-a`, plus the project slug for the project shape.
  - `ShapeBeam` goes through the same project-record write, but the review named only the tapestry, tasks and project shapes. That is not a gap in the fix.
- **F3:** `decisionVerb` has one `--from` path, through `decisionEntries`, and it sets `pattern_withdrawn` for every entry. The new row exercises it.
- **F4:** All three named branches now have a row.
  - A PATTERN changes section before `## Decisions`.
  - A bullet on an unterminated last line.
  - A fenced `Withdraws:` line.
- **F5:** `raddleBoardContext` has one missing-`PATTERN.md` branch (`internal/loomcli/raddlecontext.go:48-51`), and the new check covers it.

Tool calls:
1. `grep` for `writeRecords` in `internal/loomshed/raddlefile.go`: it matched nothing because I passed the wrong pattern, so I did not see the function and did not know which records it writes.
2. `grep` for `PatternClusters`/`pattern_clusters` across `internal`: to find every user of the key, which my context lacked.
3. `grep` for `PatternClusters`, `Project.Slug` and `PartRecordPath` in `internal/loomshed/raddlefile.go`: to find the lines of `writeRecords`.
4. `Read` of `internal/loomshed/raddlefile.go:268-317`: to confirm which slugs `writeRecords` writes records for.
