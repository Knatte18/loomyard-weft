# Plan round 2 — fixer report

## Fixed

### F1 — a verify-nested-module test row contradicts the check's own rule

Validated: with `cd nested`, `-C ../other` puts the command in `other` and `./pkg` in `other/pkg`, so both sides have module `other` and the row is silent under card 3's rule.

Change, card 3:
- The fixture worktree holds `nested/go.mod` and `other/go.mod`.
- New row `cd nested && go -C .. test ./nested/pkg`, which fires only because `-C ..` resolves against the `cd`.
- The `cd nested && go -C ../other test ./pkg` row stays, now expected silent.
- The `-C` rule states that a `-C` to an absolute path or out of the root ends the chain's scan, as a `cd` does, so the root-resolved reading of the new row has a defined outcome.

### F2 — card 5 leaves the merge of typed references and scan matches unstated

Validated: the re-signed `scanReferences` takes no `unresolved` argument, and the card did not say where the line filter applies or how the two per-subject maps combine.

Change, card 5:
- `callerCoverageFindings` combines the two per subject and file: it drops a checked file's scan matches outside that subject's `unresolved` lines; for a package-level subject it merges typed and kept scan lines of one file into the one blocking `uncoveredFinding`; for a method subject it drops a kept scan line `typedReferences` already counts.
- The existing `undefined().Method()` row of `TestGlyphChain_CallerCoverageTyped` gains a line holding that call beside a typed `t.Method()`, reported blocking and absent from the collapsed informational finding.
  This extends an existing row, so no new top-level test and no budget change.

### F3 — card 4 leaves `scanReferences`' doc comment stating the deleted rules

Change, card 4: `internal/planglyph#scanReferences` joins the `Edit:` group with no arrow, and the docs sentence says its doc comment replaces the package-clause and method rules with the import-path rule and the import-name exclusion.

`lyx loom validate-plan` passes after the changes, with only the informational `create-new-unit` findings.

## Disputed

None.

## Deferred

None.
