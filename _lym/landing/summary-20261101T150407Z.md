# Add lym's design invariants as PATTERN entries

Six invariants of lym's code existed only as rules in the orch's own note, so no writer, reviewer or judge saw them.
Every one of those agents reads `PATTERN.md` in full, so each invariant is now an entry there.

## What changes

- A new `## Code` section at the top of `PATTERN.md` holds three entries that apply across every package:
  - `PATTERN-doc-go-documentation`: a `doc.go` documents its package; a rule other code must follow is a PATTERN entry, never a `doc.go` paragraph.
  - `PATTERN-named-constants`: a name or path that can go stale is one named constant, never a repeated literal.
  - `PATTERN-no-hard-coded-choice`: lym hard-codes no language, engine or branch; each comes from config, the repository's git state or a table row keyed by name.
- `PATTERN-fabric-isolation`, under Fabric and git: fabric learns nothing of IDEs, shuttle, the board or heddle, and offers only general seams.
- `PATTERN-shed-shared-machinery`, under Shed and loom: what every phase machine needs goes once into shed's packages.
- `PATTERN-lym-instructions-in-code`, under Agents and prompts: lym's own instructions to agents live in its stencils and specs, and heddle holds only a repository's own instructions.
- Two background files carry what the one-line entries cannot:
  `pattern/PATTERN-fabric-isolation.md` defines a general seam,
  and `pattern/PATTERN-no-hard-coded-choice.md` says where each of the three choices is read from.

## Design decisions

- Every new entry is held by review, so none carries `(test)`.
- Fabric and some language and branch call sites already break the rules they now state: fabric imports packages of the board, the IDE, heddle, reed and webster, and the plan parser reads an absent language as `go`.
  The background files call these existing literals and imports debt that a change touching them moves out and never extends, so the entries bind new code without forcing a sweep in this change.
- The heddle rule is its own entry rather than a clause on `PATTERN-heddle-instructions`, because that entry governs how heddle's store is rendered, and this one governs what goes into it.
- The overview grows by about 1.1 KB and stays well under the headroom line; `lym loom lint-pattern` passes.
