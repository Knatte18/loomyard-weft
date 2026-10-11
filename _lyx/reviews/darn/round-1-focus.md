---
round: 1
focus:
  - "internal/battenshed/notice.go renderNotice: does every path in a notice (condition, child reason, stop report clause) render relative to the pair, including paths that fall outside the pair or are already relative?"
  - "internal/battenshed/notice.go: is `error:` used only for failed and blocked children and `waiting:` for every other state, and does the empty-reason case still read correctly?"
  - "Board entry's 'update the docs that quote a notice line': do doc.go and any other doc or testdata quoting a notice line match the new shape, and does anything still quote the old `child left running` or attach text?"
  - "PairDir rename from AttachDir: is any use of the old name left (code, tests, docs, envkit), and does the rename stay within what the entry licenses?"
  - "Test economy: does notice_test.go's TestRenderNotice cover each new behavior once without duplicating the lifecycle integration tests, and is the driver-busy notice's new leading state text covered?"
  - "Summary claims: does the diff bear out the example notice lines and the driver-busy claim in _lyx/landing/summary.md?"
---

Seeded from the board entry's four wanted items and the diff's file list.
