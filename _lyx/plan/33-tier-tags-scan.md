# Card 33 — tier-tags-scan

**Create:**
- `plan:cmd/lyx#TestTierTags_OneTagPerTestFile` -> `func TestTierTags_OneTagPerTestFile(t *testing.T)`
- `plan:cmd/lyx#tierTagFailure` -> `func tierTagFailure(rel string, data []byte) string`

**Edit:**
- `PATTERN.md`
- `pattern/PATTERN-test-speed.md`

**Uses:**
- `internal/testkit/scankit#`

**Intent:**
Enforce the one-tag rule over the repository (D5), right after the last retagging card, since the scan passes the repository only once every test file is retagged.

`TestTierTags_OneTagPerTestFile` is a `//lyx:guard` test in a new `cmd/lyx` test file tagged `tier0 || integration` (D13), walking every `_test.go` file through `scankit.Walk` (testdata directories excluded) with a scan floor.
`tierTagFailure` parses a file with `go/parser` and fails it, naming the file and the way forward of giving it one `tierN` tag:
when it holds a `Test`, `Benchmark`, `Fuzz` or `Example` function other than `TestMain` and its constraint names no `tierN` tag;
when its constraint admits two tiers, such as `tier2 || tier3`, and it holds such a function;
and when its constraint carries the transitional `|| integration` term while the file holds no `//lyx:guard` marker.
A function inside a string literal, such as testtiming's fixture source, is no function of the file, since only the parsed declarations count.
Its table rows feed `tierTagFailure` fixture sources: a test function in an untagged file fails; one in a `tier2 || tier3` file fails; `tier0 || integration` on a guard-marked file passes and on an unmarked one fails; an untagged file holding only `TestMain` and helpers passes; a `tier2 && !windows` file passes.
The walk over the repository passes.

`PATTERN.md` gains this line under Testing, after `PATTERN-test-speed`:

```
- `PATTERN-tier-tags` — Tagging a test file: one `tierN` build tag, the tier that runs it; an untagged test file holds no test. (test)
```

`PATTERN-test-speed.md` names this test as the rule's enforcement.

This card also files the D13 follow-up on the board with `lyx board upsert`, as a task with the slug `drop-transitional-tag-spellings` depending on this task's own entry, `gate-runs-once`, which the card's Verify line checks is on the board before the card lands: after the deploy, remove the guard files' `|| integration` term, this scan's allowance for it, the old tag spellings in the hub's tier commands, in card 11's package loads, in `cmd/testtiming`'s spawn scan and in card 12's two guards, and the old tags in card 16's purity guard.
It writes another entry and changes nothing of this task's own entry.

**ImpactSummary:** Adds one repository-scanning guard test and one overview line, and files one board entry; no production code changes.

**Verify:** lyx board find drop-transitional-tag-spellings | grep -q '"slug": *"drop-transitional-tag-spellings"'
