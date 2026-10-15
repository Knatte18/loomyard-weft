# Webster deviations list no longer mixes in its own target files

Batch 01 (`01-planparser-ref-member`) ran as one batch of four cards and recorded `done` at head `332dd16ae9d04d22d4333a563c1c57bb574bd68a`.

- `planparser` exports `RefMember`, with a nil plan reading as the default language.
- `planglyph.ScopeGuard` treats file and package targets as covering the symbols in their extent; a Modified symbol with no `Before` or `After` file is still reported.
- The implementer body's per-card step 2 states one deviation rule, and the guard-step paragraph points at it.
- The plan spec's deviation paragraph points at `planglyph.ScopeGuard`.

The digest carried no `deviations` field entries and no warnings, so the scope guard found no file changed outside the batch's targets.
