# Card 10 — live check of Claude Code's bracketed-paste behaviour

**Create:**
- `internal/shuttlecli/smoke_paste_test.go`

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/reedengine#Engine.PasteSubmit`
- `plan:internal/shuttleengine/claudeengine#Claude.PasteSubmits`
- `plan:internal/shuttleengine/claudeengine#Claude.OwnPastePlaceholder`
- `plan:internal/shuttleengine/claudeengine#PasteCollapseMinChars`
- `internal/shuttleengine/claudeengine#Claude.InputBoxText`

**Intent:**
Pin, against a real Claude Code in a real tmux pane, the facts the paste path rests on (assumption A7, risk R1), so claudeengine's choice of D2 variant is backed by evidence the operator can rerun.

`smoke_paste_test.go` carries `//go:build llm` and follows `smoke_interrupt_test.go`'s setup: a hub from `hubforge`, the package's `tmuxkit` isolation, `llmkit`, `smokeClaudeModel`, and a shuttle run started through `shuttleengine.NewRunner` with `claudeengine.New()` whose prompt asks the agent to wait for the next message.
Once the run is ready, the test drives reed's `PasteSubmit` directly and reads the pane through `Claude.InputBoxText`:
- a plain one-line text pasted with `submit` true starts a turn (a turn start appears in the run's events file) and leaves the box empty;
- `/color green` pasted with `submit` true leaves the box empty and starts no turn;
- a text of `claudeengine.PasteCollapseMinChars` characters, sized from the constant and never a literal, pasted with `submit` false shows a box that `OwnPastePlaceholder` accepts for that text, and one character shorter shows the text itself.
It asserts that what it observed for the first two equals `Claude.PasteSubmits()`, so a Claude Code release that turns the post-paste Enter into a newline fails here and names the constant to flip, and tears the strand down on cleanup.

The test is tier 3: the plan's `## verify:` compiles it (`go vet -tags llm`) and never runs it, and the operator runs it by hand when Claude Code changes; the implementer does not run it.
Raise `internal/shuttlecli`'s row in `cmd/lyx/testdata/test-budget.yaml` by the new top-level test.

**ImpactSummary:** A new llm-tier test file and one budget row; no production code changes.
