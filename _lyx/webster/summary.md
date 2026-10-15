# Webster kick-start pack layout switch

Batch `01-config-key` was begun, forked, and recorded `done` at head `611b9b44cd6cda49e5da636940ec541821d322b1`.
It is the only batch in the plan.

The batch covers the plan's four cards: the `kickstart_pack_file` config key, the three inline-layout fragments, the body's inline marker with `RenderForkPrompt` and begin-batch passing the key, and `pack_layout` on the `webster: kickstart pack` trace line.

The digest lists one deviation: `internal/websterengine/template_test.go` changed outside the batch's declared targets.
That is informational, and `record-batch` raised no warnings.
