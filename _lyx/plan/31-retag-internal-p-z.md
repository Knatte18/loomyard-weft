# Card 31 — retag-internal-p-z

**Edit:**
- `internal/pairteardown#`
- `internal/parentdirective#`
- `internal/parentlock#`
- `internal/parentreview#`
- `internal/pattern#`
- `internal/pauserecord#`
- `internal/preflight#`
- `internal/preflightshed#`
- `internal/proc#`
- `internal/quarrycli#`
- `internal/reportcli#`
- `internal/reportengine#`
- `internal/reviewbudget#`
- `internal/runproc#`
- `internal/runprofile#`
- `internal/seatengine#`
- `internal/segmentcolor#`
- `internal/selfexec#`
- `internal/selfreportcli#`
- `internal/selfreportengine#`
- `internal/shell#`
- `internal/shellguard#`
- `internal/shortcutcli#`
- `internal/sleepjournal#`
- `internal/standalonegeom#`
- `internal/standalonestate#`
- `internal/state#`
- `internal/statuscommit#`
- `internal/statuspane#`
- `internal/stencil#`
- `internal/stencilcli#`
- `internal/stencilstore#`
- `internal/summaryparser#`
- `internal/tendercli#`
- `internal/tenderengine#`
- `internal/termwindow#`
- `internal/toolfloor#`
- `internal/treadleengine#`
- `internal/twilldirective#`
- `internal/verifyrun#`
- `internal/verifytree#`
- `internal/vscode#`
- `internal/weftname#`
- `internal/yamlengine#`
- `PATTERN.md`
- `pattern/PATTERN-test-speed.md`
- `crucible/README.md`
- `crucible/orchestrator-prompt.md`
- `crucible/review-prompt-template.md`

**Create:**
- `cmd/lyx#TestTierTags_OneTagPerTestFile`
- `cmd/lyx#tierTagFailure`

**Uses:**
- `internal/testkit/scankit#`

**Intent:**
Retag every test file of these packages to its one tier tag (D5), under the overview's `retag-rules` decision, as card 23 states the rules; this is the last retagging card, so it also lands the one-tag scan (D5), which passes the repository only once every test file is retagged.
The crucible docs' live-substrate commands, found by a search for `-tags` over the non-Go files, compile the retagged tests only under the new spelling, so `go test -tags tmux` becomes `go test -tags tier2` and `-tags llm` becomes `-tags tier3` in `crucible/README.md`, `crucible/orchestrator-prompt.md` and `crucible/review-prompt-template.md`.
The `//go:build tmux` and `//go:build llm` prose of `crucible/README.md` and `crucible/review-prompt-template.md`, which tells a writer which tag a live test carries, names `tier2` and `tier3`, since the one-tag scan this card lands refuses a test file tagged `tmux` or `llm` alone.

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

`PATTERN.md` already sits above its headroom line, so this card's `lint-pattern` step passes only when the overview is no longer than at HEAD.
In the same edit, these eight entries take the shortened wording the parent's `changes` decisions of 2026-10-24 record, verbatim, which saves at least the new line's length; no rule changes:

```
- `PATTERN-lyxdirs-single-declarer` — Naming `_lyx` or `.lyx` in a path: only `internal/lyxdirs`, bar a `tools/` script's variable block.
- `PATTERN-treadle-runner-seam` — Importing into `internal/treadleengine`: allowlist only, never `burlerengine`, `lyxcwd` or a CLI package.
- `PATTERN-shed-producer-seam` — Importing into `internal/shedengine`: only stdlib, `state` and `lock`; paths are told.
- `PATTERN-parent-lock` — Moving, merging into or landing on a local parent branch: only in Finalize, under its parent lock.
- `PATTERN-fabric-write-containment` — Writing `_launchers` or workspace file in `fabricengine`: via a hub `os.Root` only.
- `PATTERN-origin-reconcile` — Acting on a run or hub after an absence: `fabricengine`'s origin reconcile first.
- `PATTERN-hub-store-gc` — Hub stores run no auto gc: only `fabricengine/housekeeping.go` sets it or runs `gc`. — [background](pattern/PATTERN-hub-store-gc.md)
- `PATTERN-owner-machine` — Starting, resuming or landing a run or acting on wake: only where no other machine owns it. — [background](pattern/PATTERN-owner-machine.md)
```

`PATTERN-test-speed.md` names this test as the rule's enforcement.

This card also files the D13 follow-up on the board with `lyx board upsert`, as an entry of kind `task` carrying the type label `enhancement`, the one this task's own entry carries, with the slug `drop-transitional-tag-spellings` depending on this task's own entry, `gate-runs-once`, which the card's Verify line checks is on the board before the card lands: after the deploy, remove the guard files' `|| integration` term, this scan's allowance for it, the old tag spellings in the hub's tier commands, in card 11's package loads, in `cmd/testtiming`'s spawn scan, in card 13's `defaultRedundancyTags` and in card 12's two guards, and the old tags in card 16's purity guard and in its tmux isolation guard's `isolationTags` and `isolationTagSets`, and the `tmux` and `llm` terms of `internal/testkit/tmuxkit/allow_tmux.go`'s constraint, and respell planglyph's `testdata/glyphchain/shapes/tagged.go` fixture from `integration` to `tier1`.
It writes another entry and changes nothing of this task's own entry.

**ImpactSummary:** Changes only the build-constraint lines of these packages' test files, splitting a shared helper file where another tier uses it, adds one repository-scanning guard test and one overview line, rewords eight overview lines with no rule change, respells the crucible docs' test tags, and files one board entry; no production code or caller changes.

**Verify:** lyx board find drop-transitional-tag-spellings | grep -q '"slug": *"drop-transitional-tag-spellings"'
