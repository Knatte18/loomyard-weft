# Card 7 — Pin tolerant run records in PATTERN

**Create:**
- `pattern/PATTERN-tolerant-records.md`
- `cmd/lyx/tolerantrecords_test.go`
- `plan:cmd/lyx#TestTolerantRecords_NoStrictJSONDecode` -> `func TestTolerantRecords_NoStrictJSONDecode(t *testing.T)`
- `plan:cmd/lyx#strictDecodeFindings` -> `func strictDecodeFindings(filename string, data []byte) ([]string, error)`

**Edit:**
- `PATTERN.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
D9: a PATTERN entry pins that no reader of a Go-written JSON record refuses an unknown key, and that a rewrite keeps them.
Cards 2, 5 and 6 removed every production `DisallowUnknownFields` call, so the scan this card adds is green on landing.

`PATTERN.md` gains this line, under `## Shed and loom` after `PATTERN-shed-run-directory`:

```markdown
- `PATTERN-tolerant-records` — Go-written JSON records: never refuse an unknown key; rewrite via `internal/state`. (test) — [background](pattern/PATTERN-tolerant-records.md)
```

The overview already sits past its headroom line, so the branch may not grow it: `internal/pattern#Headroom` reports any growth above the line.
Four entry lines shrink to pay for the new one, each cutting only detail its background file already states (checked: `<BoardDir>/_lyx/config/` and `LocalKeys` in `pattern/PATTERN-hub-wide-config.md`, `host` retired in `pattern/PATTERN-fabric-vocabulary.md`, rebase-free and never by an agent in `pattern/PATTERN-push-both-sides.md`, the pre-write guard, the activity check and the Enter rule in `pattern/PATTERN-pane-text-injection.md`):

```markdown
- `PATTERN-hub-wide-config` — A hub-level config module: `HubWide` in `configreg`, its file on the board only. — [background](pattern/PATTERN-hub-wide-config.md)
- `PATTERN-fabric-vocabulary` — Naming the wired composite: "fabric"; warp and weft only to tell its sides apart. (test) — [background](pattern/PATTERN-fabric-vocabulary.md)
- `PATTERN-push-both-sides` — Pushing run records from Go: the pair's code branch goes along; the prime sends records only. — [background](pattern/PATTERN-push-both-sides.md)
- `PATTERN-pane-text-injection` — Typing text into an agent pane: only reed's guarded paste-submit. — [background](pattern/PATTERN-pane-text-injection.md)
```

Together they cut 285 bytes and the new line adds 178, so the overview ends smaller than its base; the card's verify runs the headroom lint from the card's parent commit.

`pattern/PATTERN-tolerant-records.md` holds the detail: no reader of a Go-written JSON record refuses an unknown key; a rewrite goes through `internal/state`, which keeps the keys its type does not know (the merge rules and their bound, pointing at `internal/state`'s package doc rather than restating them), or through a type that keeps them itself, as `board.json`'s `storeFile` and `Task` do; a read-only decode outside `state`, such as batten cleanup's judge of committed bytes, is allowed; agent-written contract files (webster's `outcome.yaml` and `report.yaml`, the arbiter's `verdict.yaml`, plan and recipe files) stay strict, since no binary rewrites them; the scan covers only `encoding/json`'s `DisallowUnknownFields`, so `yaml.v3`'s `KnownFields` is outside it, and a future agent-written JSON contract that needs strictness validates its keys some other way.

`TestTolerantRecords_NoStrictJSONDecode` enforces it, in the shape of the `cmd/lyx` scans beside it, with `//lyx:guard` above it.
`strictDecodeFindings` parses a file and reports one finding per call expression whose selector is `DisallowUnknownFields`, with its position, so a comment or a string naming the method is never a finding.
The test first runs it over two in-memory sources, one with a `json.NewDecoder(r).DisallowUnknownFields()` call, wanting exactly one finding, and one naming the method only in a comment, wanting none, so the scan's own failure path is reached.
It then walks every production `.go` file of the module through `scankit.Walk`, requires a scan floor, and fails naming each finding with the way forward: decode tolerantly, and rewrite through `internal/state`.
`cmd/lyx`'s budget row rises from 78 to 79.

**ImpactSummary:** the overview gains one entry and four shorter ones, under its size at the merge base; one new scan test in `cmd/lyx`.

**Commit:** 7: PATTERN-tolerant-records and its scan

**Verify:**
- `lyx loom lint-pattern --base HEAD~1`
