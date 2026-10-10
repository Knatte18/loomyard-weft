# Fabric-reference audit fails a batch for help-only lyx fabric --help calls

`fabric-audit-help-false-positive` · Note · bug, webster, fabric, stencil

Imported from [issue #514](https://github.com/Knatte18/loomyard/issues/514).

## Seen

In run `cli-surface` (2026-10-10), batch 06's implementer fork read help text with `for v in ...; do lyx fabric $v --help --json | jq -r .long | wc -c; done`.
The fabric-reference audit (`ClassifyViolation`, `internal/websterengine/audit.go:384-396`) classed it as a correctness violation, and `fabricengine.IsReadOnlyCommand` (`internal/fabricengine/refscanner.go:184-229`) rejects it, so `accept-audit` could not keep the batch's six committed cards.

## Workaround

`lyx webster reset --to batch-start --batch 6`, `lyx webster accept-audit --batch 6`, then redo the batch, discarding cards 6-11.

## Expected

A `lyx ... --help`, `-h` or `--json` invocation (none runs a command since 57243019b) is not a fabric reference, or the implementer stencil forbids running `lyx` at all so forks read code instead.
