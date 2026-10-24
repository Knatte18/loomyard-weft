---
verdict: APPROVED
findings:
  - id: F1
    severity: NIT
    class: consistency
    location: cmd/lyx/gitrepoboundary_test.go:13
    summary: The comment puts a prose clause inside the backticks that mark the ID, so the span reads as one identifier
    fix: In the comment at line 13, change "`PATTERN-gitrepo-client-boundary's checked-call rule`" to "`PATTERN-gitrepo-client-boundary`'s checked-call rule".
    origin: handler
---

### [NIT] Backtick span swallows prose in a rewritten citation

**Location:** cmd/lyx/gitrepoboundary_test.go:13
**Issue:** The sweep rewrote `PATTERN-gitexec-checked-call` to the survivor's ID, and the comment now reads `` `PATTERN-gitrepo-client-boundary's checked-call rule` ``.
Every other rewritten citation in the diff backticks the ID alone, as `internal/gitexec/gitexec.go:19` and `internal/gitrepo/remote.go:19` do without backticks.
No behavior is affected.

## Rejected

- code-tests lens, LOW: the `cmd/testtiming/redundancy.go` footer is not pinned by a test.
  The lens marked it weak, and `grep` over `cmd/testtiming` finds no assertion on the string, old or new.
  The target section "Test economy" exists in `pattern/PATTERN-test-speed.md`, so the retargeted pointer is correct.

## Amendment verdicts against the focus directive

- Merged test content: `git diff ed98ccf71 93f1c8942` on the four parent-touched pattern files shows every change present in `PATTERN-test-speed.md` and `PATTERN-test-isolation.md`, with nothing dropped: the `tier0` through `tier3` spellings, the `//lyx:guard` clause that refuses `integration`, `tmux` and `llm`, the `tier2` "no coverage" wording, and the `tier3` testkit line.
  The 18 parent-touched `.go` files cite only surviving IDs or package docs.
- Cited IDs: a sweep of every `PATTERN-` token outside `docs/research/`, `docs/benchmarks/`, `testdata/` and `_lyx` finds only IDs in `PATTERN.md` and test placeholders (`PATTERN-a`, `PATTERN-x`, `PATTERN-old-rule` and similar).
  The parent's `gate-runs-once` tier-tag and verified-tree tests cite `PATTERN-test-speed` and `PATTERN-gate-slot`.
- `pattern.Check` mechanics: 30 entries, 28 background files matched one to one, with `github-auth` and `shell-guard-table` the entries without one.
  `lyx gate test ./internal/pattern` passes.
- Moved rules: the `tools/sandbox/doc.go` and `internal/lyxcwd/lyxcwd.go` sections keep the meaning of the original rule; for `hub-suffix` the original background file was compared clause by clause.
  The merge groups match the board entry, with the `lyxdirs-single-declarer` `tools/` carve-out kept; the three placements the decision record declares (`hub-store-gc`, `row-entry-gate`, `gate-carry`, `tier-tags`) are the only departures from the board entry's lists.
- Size: `PATTERN.md` is 6966 bytes against a 16384-byte cap, well under its headroom line.
