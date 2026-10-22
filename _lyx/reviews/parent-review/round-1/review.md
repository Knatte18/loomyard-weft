---
verdict: APPROVED
---

## Scope check

The board entry `lenses-language-sets` carries no Done-when list; its operator decision of 2026-10-22 is the binding goal:

- A shared base of the lenses that are the same for every language: D1 makes `internal/lenses/template.yaml` language-neutral.
- A predefined language layer per language that replaces the few lenses that speak to the language: D2 ships embedded `go`, `csharp` and `python` layers, each replacing the five language lenses, and D3 picks one with `lens_set`.
- Each repository can still add its own lenses: the decision record keeps add and override by name (Goal, acceptance criterion on hub and worktree entries beating the layer).
- Each repository can remove lenses it does not want: D5 adds `remove`.

The entry's C# examples, nullable and `IDisposable` handling and numerical stability and units, are in D2's `csharp` layer and its `numerics` lens.
The operator's note that a rules lens must read a hub's rules where the hub keeps them is met by D1's rewording of `pattern-fit`, `code-pattern-fit` and `docs-consistency`.
The design keeps those rules lenses in the base rather than in the layers, against the entry's candidate list; that list sits under "Orch recommendation, not a decision", and the record states the departure under Auto-mode assumptions, so it is no scope deviation.
The entry rules out language logic in code, and D6 and the Scope Out list detect no language from repository files.
No work in the design belongs to another entry: `review-setup-audit` audits the review setup and builds no lens layer, and `dotnet-verify-protocol`, `dotnet-scoped-verify` and `regression-baseline-verify` touch no lens.

## Overlap with peers and parent commits

`lyx loom peers` lists `own_row` 3 and no parent commits.

- `machine-handover` (row 6): owner machines, `lyx adopt` and `ReconcileOrigin`; no shared behaviour, package or PATTERN entry. No overlap.
- `quarry-csharp` (row 7): C# glyphs in `planparser` and `planglyph`; its Scope Out names `lenses-language-sets` explicitly. No overlap.
- `darn-verify-fixes` (row 10): Darn-Write's verify-failure routing in `landingshed`, `loomshed` and `shedrun`; no shared behaviour or PATTERN entry. No overlap.

No peer adds or changes `PATTERN-template-overlay`, and none touches `internal/lenses`, `internal/configengine` or `internal/configsync`.
With no overlap, no landing order applies.
