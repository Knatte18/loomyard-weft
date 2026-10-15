# webster: per-card kick-start pack layout, on by default

The Webster implementer fork gets a third kick-start pack layout.
Instead of reading a batch-wide pack up front, the fork reads every card file of its batch first as an overview, with no code.
Before each card's first edit it fetches that card's pack and reads the listed spans in one parallel turn.
The audit of the file layout showed forks reaching only about a quarter of a batch pack's spans before their first edit, and line numbers go stale after the first edit anyway.
Resolving per card against the tree as it stands gives current spans and a smaller read.

The board entry for this task closes no GitHub issue, so there is no `Closes` line.

## What changes

- `webster.yaml` gains the bool `kickstart_pack_per_card`, default `true`.
  Layout resolution at begin-batch: `kickstart_pack: false`, a glyph-less language or an empty batch pack gives no pack, as before; otherwise `kickstart_pack_per_card: true` gives the per-card layout, and only when it is false does `kickstart_pack_file` choose between the file and inline layouts.
  The default means every run uses the per-card layout after deploy.
  `lyx config webster --set kickstart_pack_per_card=false` restores the file layout.
- New read-only verb `lyx loom card-pack <card>`, audience `role`, hosted in `internal/loomcli`.
  It parses the plan afresh, resolves the one card through `planindex` against the worktree on disk (committed or not), and prints the card's subsection in the batch pack's format as plain markdown on stdout.
  A card with no member glyph, or a glyph-less language, prints the empty-card line.
  A quarry failure prints every glyph unresolved with the error, exit 0.
  A non-numeric argument, an unknown card, a missing plan directory and an unparsable plan are refused, each with a `refusal-spec.md` row whose way forward is to locate the glyphs with `lyx quarry resolve` or a grep.
  It logs one `webster: card pack` Info line (card, resolved, unresolved) when it renders a pack, and mutates no state.
- Fork prompt: new fragments for the per-card FRESH-READ rule, step 1 of each card, the locate step, a closing `## Start here` section, and a `lyx loom card-pack` entry in the sandbox rule's allowed-command list.
  Each is registered in `contracts/stencils/stencils.go` and `.gitattributes`.
- begin-batch under the per-card layout still builds the batch pack, only to decide empty and to count, but renders no pack into the prompt and writes no `<batch>-pack.md`.
  The `webster: kickstart pack` line ends `pack_layout=card`.
- `tools/forkaudit` learns the card arm: a marker row matching `lyx loom card-pack`, checked ahead of the file row, the `card` trace value, spans read from `card-pack` tool results with the `webster: card pack` lines as cross-check, and a card-traces pass in `forkaudit.sh`.
- Docs: `internal/websterengine/doc.go`, `internal/loomcli/doc.go` and `docs/overview.md`.

## Design decisions

- The verb lives under `lyx loom`, not `lyx webster`, because the `steerWebsterForkDeny` hook refuses every `lyx webster` command from a fork.
  The engine function is exported from `websterengine` and handed the `planglyph` index by the CLI, so the engine imports neither `planglyph` nor quarry.
- The batch overview is the card files themselves; no overview is rendered.
- The file layout, inline layout, pack-off prompt and recovery prompt render the bytes they did before, except for the fragment files below.
  The sandbox paragraph gains a new optional marker that is empty everywhere but the per-card layout.
- Span code inside a pack was rejected: it goes stale, and an Edit needs the file read by the fork anyway.
- `exported websterengine.CardPackWayForward` holds the refusal way-forward text shared by the verb and its tests.

## To check by hand

- Several fragment files outside the new ones changed by a line or two (`webster-fragment-locate-pack.md`, `-locate-pack-inline.md`, `-fresh-read-pack-inline.md`, `-card-read-pack.md`, `-pack-card.md`, `-pack-empty.md`, `-pack-read.md`, `-pack-section-inline.md`, `-pack-unresolved.md`).
  Confirm the file and inline layouts render the same bytes as before; the acceptance criterion is byte equality.
- Default on changes every run's fork prompt on the next deploy, and a run already in Webster switches layout at its next begin-batch, so one run can mix layouts.
  The audit classifies per batch through the begin-batch `batch` key.
- `tools/forkaudit` card arm, window bias: the card arm's spans window is open as a MEDIUM finding that it may be biased, in `tools/forkaudit/extract.jq` and `report.jq`.
- `tools/forkaudit` card arm, missed files: later cards' pack files missing from the missed-files count is open as a BLOCKING-class finding, in `tools/forkaudit/pack-entries.jq` and `report.jq`.
  Check that a per-card fork's missed files are counted correctly before relying on its numbers.
- The measurement against the file layout was not run.
- The task `lighter-test-rules` (Ready) will relax wording tests; whichever lands second adapts.
