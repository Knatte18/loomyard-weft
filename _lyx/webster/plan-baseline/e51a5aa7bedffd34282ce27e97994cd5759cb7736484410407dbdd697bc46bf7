# Card 6 — pause-conditions

**Edit:**
- `internal/shedverbs#pauseCmd`
- `internal/shedverbs/pause_test.go`
- `internal/shedverbs/doc.go`
- `internal/shedcli#shedVerbTexts`
- `internal/loomcli#loomVerbTexts`
- `contracts/specs/refusal-spec.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `plan:internal/shedverbs#pauseFlags` -> `type pauseFlags struct`
- `plan:internal/shedverbs#applyPauseConditions` -> `func applyPauseConditions(cur shedengine.Status, flags pauseFlags, producers []shedengine.ProducerDef) (shedengine.Status, map[string]any, error)`

**Uses:**
- `internal/shedengine#Status`
- `internal/shedengine#Routing`

**Intent:** The verb half of D3: `lyx shed pause <run-id> --before <producer>`, `--after <producer>` and `--clear`, and so `lyx loom pause`, since the flags land on the generic command.
`pauseCmd` registers the three flags into a `pauseFlags` value and keeps its single `state.UpdateJSON` write under the status lock; the mutate calls `applyPauseConditions`, a pure function tested through `pauseCmd`.
With no flag the verb keeps its meaning and sets `pause_requested`.
`--before` or `--after` records the condition in its slot and leaves `pause_requested` alone; a slot already set is replaced, and the answer names the replaced target under `replaced_before` or `replaced_after`.
`--clear` removes both conditions whoever set them, names the removed targets under `removed_before` and `removed_after`, and leaves `pause_requested` alone, so an operator's bare request stays recorded and Shed remains its only writer of `false`.
`--clear` beside `--before` or `--after` is refused as contradictory with the message `shedverbs: pause --clear cannot be combined with --before or --after; way forward: run pause --clear alone, then pause --before or --after`, and the file is untouched.
A target must name a producer of `spec.Routing.Producers`; another name is refused with the message `shedverbs: pause target <name> names no producer; way forward: re-run pause with --before or --after naming one of: <names>`, and the file is untouched.
The success envelope carries `status_file`, `before` and `after` as recorded, plus the replaced and removed keys when non-empty.
`pause_test.go` gains `TestPauseCmd_Conditions`, a table driven through `pauseCmd` with `execEnvelope`, asserting each row's envelope and the status file re-read after the verb: set before, set after, replace names the replaced target, clear removes both and leaves a recorded bare request, clear with a condition flag refused with its way forward and the file unchanged, unknown target refused listing the producers and the file unchanged.
`shedVerbTexts` and `loomVerbTexts` describe the flags in the pause verb's `Long`.
`refusal-spec.md`'s shed section gains the row `pause target unknown` (correctness halt) with that way forward, and the row `pause flags contradictory` (correctness halt) with its way forward.
`doc.go` names the pause flags.
`test-budget.yaml` raises `internal/shedverbs` from 65 to 66.

**ImpactSummary:** `pauseCmd` keeps its envelope on the bare path; the two `VerbTexts` edits are help text only.
