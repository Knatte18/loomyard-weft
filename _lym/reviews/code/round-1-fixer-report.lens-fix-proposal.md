```yaml
lens: fix-proposal
outcome: reported
```

Fix-proposal lens: I challenged the proposals of F1, F2 and F3, which were all fixed as proposed. I found one LOW finding.

**LOW — `docsLinkIsResearchData` (`internal/lymcwd/docslink_test.go:316`)**
- **Changed text:** `len(parts) > 5 && ... slices.Contains(parts[4:len(parts)-1], "data")`.
- **Summary:** The proposal handles a topic-level `data/` folder but not a series-level one, so the same boundary stays open one level down.
- **Evidence:**
  - A series folder such as `docs/research/codex/v2/` or `docs/research/codex/astra/` is not a study, just as a topic is not.
  - `docs/research/topic/series/data/x.md` splits into `docs, research, topic, series, data, x.md`. `parts[4:5]` is `[data]`, so the predicate skips it.
  - The new doc comment says a `data/` folder directly under a topic "belongs to no study and is scanned". The code applies that only to topic level.
  - No such folder exists today. Every `data/` folder under `docs/research` sits at study depth, directly under a study folder (`.../<study>/data`).
  - The `parts[4:]` bound can't tell a series folder from a study folder without reading the tree. The predicate would have to check what sits above the `data/` folder, or the test would have to pin the series-level skip as intended.
- **Why only LOW:** the finding asked for the topic-level boundary only, and no live file is affected.

**F1 and F2:** no findings.
- **F1:** `report.md` exists beside `deviations.md`, and `twill-discussion-bench.md` is gone from that folder.
- **F2:** the new `docs/research/glyph/glyph-audit-2026-10-13/` holds the four `.jq` files the `cp` brace list names (`extract2.jq`, `stats2.jq`, `stats3.jq`, `stats-base.jq`). Line 101's `cat` of `README.md report.md` also resolves there.

Tool calls made:
1. `ls` on the twill bench folder and the glyph 2026-10-13 folder, plus a `find` for `data` directories at depth 3–4 under `docs/research`. My context lacked the post-move folder contents and the real `data/` depths.
