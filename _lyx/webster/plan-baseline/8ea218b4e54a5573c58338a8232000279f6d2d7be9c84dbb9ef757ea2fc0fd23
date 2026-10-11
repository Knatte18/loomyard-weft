# Card 3 — twill-lenses

**Edit:**
- `internal/burlerengine/template.yaml`
- `internal/burlerengine#TestConfigTemplate_DecodesAndIsSelfConsistent`

**Uses:**
- `internal/burlerengine#ResolveFan`

**Intent:**
Add the two lenses and the fan of decision record D8 to the embedded burler lens library.

Under `lenses:`, after `leanness`, add `premises` and `board-fit`, each phrased as emphasis and ending with the library's emphasis line, which the self-consistency test requires of every lens but `generic`:

- `premises`: focus on the record's premises: every claim it makes about existing code, config or behaviour matches what the code does, including one hop past what the record cites, such as the caller, the config key or the spawned process a cited piece leans on.
- `board-fit`: focus on the record against every other board entry: planned work it duplicates or contradicts, and a dependency it leaves unnamed.

Under `fans:`, add `twill-discussion` with the entries `premises`, `goal-scope`, `board-fit`, `pattern-fit`, `scenarios`, `leanness`, in that order and with no `generic` lens.
The fan is well under the cluster cap.

`TestConfigTemplate_DecodesAndIsSelfConsistent` raises its lens and fan counts by two and one, adds `twill-discussion` to the per-fan entry counts and to the resolved-order table, so the template still decodes strictly and the new fan resolves through `ResolveFan` in the stated order.

**ImpactSummary:** A data-only change to the embedded template; no Go code reads the new names until card 9.
