```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens. I traced each function and branch the diff adds or changes, one concrete case at a time.

**Cases run**
- `raddleparser` `vocabulary()`, labels-missing branch: with `Labels: nil` it emits one `labels-missing` finding for the part. With `Labels: []string{}` it does the same, because `len == 0`. **Correct.**
- `vocabulary()`, label-unknown loop:
  - `["mod-a","ghost"]` against `[mod-a mod-b mod-z]` gives one `label-unknown` for `ghost`. **Correct.**
  - `["feature"]`, a type label, gives `label-unknown` because `LabelVocabulary` holds only the config's `labels`. **Correct.**
  - `["mod-b"]`, which the Sketch does not carry, passes. **Correct.**
  - An empty `LabelVocabulary` (board.yaml has no `labels`) refuses every label. The message names the operator's way forward, and the Goal asks for exactly that refusal. **Correct.**
  - Duplicate `["mod-a","mod-a"]` gives no finding. **Wrong downstream; see finding 1.**
- `loomcli` `raddleBoardContext`: it fills `LabelVocabulary` from `boardConfig.Vocabulary().Labels`. A failure to load the config returns before the literal. **Correct.**
- `loomshed` `filing()`:
  - Part labels: `append([]string{part.Type}, part.Labels...)` builds a fresh slice each time, so no slice is shared between entries. With `Type: "feature", Labels: ["mod-a"]` it gives `[feature mod-a]`. **Correct.**
  - Project labels: `[feature]` plus `raddleProjectLabels(parts)`. With parts `[mod-a]` and `[mod-a mod-b]` it gives `[feature mod-a mod-b]`. With zero parts it gives `[feature]`, the nil union. **Correct.**
- `raddleProjectLabels`: with `[]`, the result is nil. With repeats, it dedupes in first-seen order across the parts in proposal order. **Correct.**
- Filing `Check` closure (`raddlefile.go:481`): `LabelVocabulary` is unset there. `RecheckBoard` (`recheck.go:28-58`) never reads it. **Correct.**
- Restart path, a proposal approved before this change with no `labels` key: Raddle-File reruns the full `CheckRaddleProposal(..., true)` (`raddlewiring.go:190`). That returns `labels-missing`, so the proposal is sent back rather than filed without module labels. **Correct.**

**Findings**

1. **MEDIUM**, class `design`, at `internal/raddleparser/check.go:431-438`.
   - Summary: the check accepts a part whose `labels` repeat a module label. The board then refuses the filed entry at Raddle-File, after approval, instead of the proposal check catching it.
   - Evidence:
     - The `vocabulary()` loop tests only membership (`check.go:434-437`).
     - Raddle-File files `append([]string{part.Type}, part.Labels...)` unchanged (`internal/loomshed/raddlefile.go:507`).
     - `boardengine.validateLabels` refuses it: "carries label %q more than once" (`internal/boardengine/labels.go:28-30`).
   - Impact: Raddle-File's filing step returns that as a board-write error, not `ErrRaddleRecheck` (`raddlefile.go:315-325`). An approved proposal halts the run instead of being refused by `raddle-propose`/the gate with a way forward.

Tool calls:
- Read `loomshed/raddlecheck.go`, `raddleparser/recheck.go` and a grep of board config `labels`. The manifest lacked the check and recheck entry points and the empty-vocabulary default.
- Grepped the callers of `CheckRaddleProposal`/`RecheckRaddleBoard` and the `raddleFile` methods. The manifest lacked whether Raddle-File reruns the full check (the restart path) and how a filing error is handled.
