---
round: 1
focus:
  - "internal/lymcwd/docslink_test.go `docsLinkIsResearchData`: does the path test match exactly `docs/research/<report>/data/...` and nothing else (a `data/` directly under a report, not deeper or elsewhere under `docs/`), including paths with exactly four parts such as a file named `data` in a report?"
  - "The new subtest: does its fixture avoid `testdata` in paths and go through `writeTree` under `t.TempDir()`, and would it fail if the skip were removed or widened to the whole of `docs/research/x/`?"
  - "Board entry clause on PATTERN text: the entry asks for the Markdown Link Integrity text in `PATTERN.md` or `pattern/` to state the scope wherever it is stated; check whether any such text exists in the repo, and whether the record's claim that none does holds."
  - "Only the source side is skipped: does the diff leave resolution of links from scanned files into `data/` files, anchors included, unchanged?"
  - "Allowlist removal: is `docsLinkAllowlist` left in a form consistent with its surrounding doc comment and with other users of `scankit.Entry`, and do other tests that reference the removed entry still hold?"
---

Small single-file change; the diff is a test-file edit, so the checks are scope against the board entry and the correctness of the path predicate.
