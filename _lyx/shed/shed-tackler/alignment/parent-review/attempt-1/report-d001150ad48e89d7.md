---
verdict: APPROVED
---

The design matches `shed-tackler`'s board entry, with no part missing and nothing added beyond it.

- Wanted, the tackler and the Go driver as shed code: D1 moves both, with their shared types, into `internal/sheddriver`.
- Wanted, darn, batten and any later recipe get them: D2's seams, D3's darn runs under `lyx loom drive`, and D4's batten run through the same body.
- Wanted, shed's config holds the defaults and a module's own config overrides them: D5's `shed.yaml` with the loom, darn and batten overrides, and the `loom.yaml` migration that keeps the hub's spelled-out defaults from masking `shed.yaml`.
- Order, the first live run on the shed version: Scope Out keeps every `driver_kind` default at `llm`.
- Stencil points: D7 makes the stencil recipe-neutral and adds the recipe marker, the stop list with usual verdicts, and the reading order.
  The entry floats the marker as optional and D7 makes it required, with a stated reason; the entry rules neither out, so this is a design detail for Discussion-Review.

No other entry is duplicated or contradicted:

- `run-verbs-shared` keeps run verbs under `lyx loom` for now and `lyx shed` to the FSM; D3 keeps `lyx loom drive` and adds no `lyx shed` verb.
- `orch-split` keeps the hub run driver and the message receiver; both are in Scope Out and Open risks.
- `loom-model-key-names` keeps the key renames; D5 keeps today's names.
- `llm-agnostic-engine` gains no new spawn site (plan-writer note).
- `shedrecipe-capabilities` and `report-timing` are unbuilt and conflict with no decision here.
- The arbiter stays loom's code and config, which the entry does not ask to move.

## Overlaps with runs in flight

`lyx loom peers` lists no landed parent commits; this run's `own_row` is 3.

- `twb-d0-sa`, `twb-d0-sc`, `twb-d0-sf` (darn decision record and PR-Align): both runs change darn's wiring (`internal/loomcli/darnwiring.go`), and a darn PR-Align row may need an entry in the per-row interrupt policy D2 tells.
  Their records already account for this run giving darn an arbiter and a moved driver; no decision contradicts one here, and D2's darn hand-back follows loom's today, as this record's Open risks says.
  Landing order: `twb-d0-sa` and `twb-d0-sc` (row 4) land first and this run takes their changes at Merge-In; `twb-d0-sf` (row 3) is a tie.
- `config-set-validate` (`lyx config --set` refuses a value the module loader rejects): it will validate the new `shed.yaml` and override keys D5 adds, through the same `CheckValues` path; complementary, no conflicting decision.
  Landing order unknown: its row is 0 and it holds no decision record.
- `twill-fork0`, `twb-d0-ga`, `twb-d0-gc`, `twb-d0-gf`: no overlap.
  The PATTERN entries they change (`PATTERN-glyph-conversion-chokepoint`, `PATTERN-gate-self-check-parity`'s background) are not ones this run changes.
