# Per-card kick-start pack layout

Batch `01-per-card-switch` ran as one fork and recorded `done` at head `5f14a3ee248c5915f6728929388fd0cb57ce79ea`.
It holds the plan's four cards, each committed on its own.

- Card 1 adds the `kickstart_pack_per_card` key to `webster.yaml`, default true.
- Card 2 adds the read-only `lyx loom card-pack` verb, which renders one card's pack against the current tree.
  The fork also exported `websterengine.CardPackWayForward`, which holds the shared refusal way-forward text.
- Card 3 renders the per-card layout in the fork prompt, picks it at `begin-batch`, and adds five stencil rows after `webster-fragment-pack-empty`.
- Card 4 teaches `tools/forkaudit` the card arm from `card-pack` results and card pack trace lines, and adds a card-traces pass to `forkaudit.sh`.

The fork reported each card gate and the batch gate green and wrote no friction note.

## Deviations and scope guard

The scope guard listed three files changed outside the batch's declared targets, informational only:

- `internal/loomcli/cardpack_test.go`
- `internal/websterengine/cardpack.go`
- `tools/forkaudit/forkaudit_integration_test.go`

The `record-batch` envelope carried no warnings.
