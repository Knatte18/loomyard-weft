# Drift: a parent decision trips the judge snapshot check, and line-range expectations never match

`drift-judge-and-line-range` · Task · bug, drift, loom · done

Two drift bugs, bundled because both fixes sit in the drift package and its loom wiring.

## Part: a parent decision during Drift-Verify halts the judge

Imported from [issue #145](https://github.com/Knatte18/loomyard/issues/145).

### Seen

While a darn run's Drift-Verify judge session was running, the parent recorded a decision with `lym loom decision add <slug> --by parent`.
The verb committed `_lym/discussion/decision-record.md` on the records side.
When the judge finished, the run halted with `decision halt: the judge session changed what it may not: records: ... decision-record.md was changed`.
The judge had not touched the file; the parent's own verb had.

### Workaround

The parent restored the record to the snapshot HEAD in the records worktree (`git checkout <snapshot head> -- .../decision-record.md`), deleted the capture's scratch directory, ran `lym loom resume`, and records the decision again after Drift-Verify ends.

### Expected

`lym loom decision add` is either refused while a drift judge session runs, naming when to retry, or its commits are excluded from the judge's snapshot check.


## Part: line-range expectation entries never match

Imported from [issue #141](https://github.com/Knatte18/loomyard/issues/141).

### Seen

In a darn run with drift scope full and `diff: lines` (key_patterns set), Drift-Expect wrote `expected.yaml` entries with `lines: - range: "1-"` or `"7-433"` per case.
The drift gate then failed on every changed value in those cases: the checker reports each changed value at line 0, so no line range ever covers it, even `"1-"`.

### Workaround

The parent amends the expectation with `lym drift amend <slug> --entries <file>` using key-based entries (`keys: - key: ..., n: ..., why: ...`), which match.

### Expected

A changed value carries its real line number so line-range entries match, or Drift-Expect is told to write key-based entries when the diff reports by key.

