# Card 21 — drift-writer-stencils

**Prosa:**
- `contracts/stencils/darn/darn-template-write.md`
- `contracts/stencils/webster/webster-template-master.md`
- `contracts/stencils/webster/webster-prefix-fork.md`

**Intent:** D16's drift paragraph in the writers' stencils, static prose with no marker change.

- `darn-template-write.md`: the sealed expectation is `_lym/drift/expected.yaml`, which the writer reads and must not edit; the `drift` gate entry checks after each turn that every changed value of the capture is covered by it and fails naming each one outside; the writer fixes such a value in code itself, and when the change is intended leaves it, since only the run's parent amends the expectation at the Stuck that follows; `lym drift check` runs the same check on demand.
- `webster-template-master.md`: Master edits no code; a failed `drift` entry at the run-level gate lands in the verify-gate report like a failed `verify`, and Master hands it to the fixer fork or `lym webster fix-verify` exactly as for any run-level gate failure (D8); a value the fixer leaves as intended keeps the entry failing, and the run's parent amends the expectation at the Stuck that follows; the expectation file's path and that no one at this row edits it.
- `webster-prefix-fork.md`: a card fork may run `lym drift check` after its card to see whether its change moved a value outside the sealed expectation, and never edits `_lym/drift/expected.yaml`.
