# Research: tree-sitter-analyzer against quarry and lyx, 2026-10-24

Adds `docs/research/tsa-quarry-2026-10-24/report.md`, a research report of ideas drawn from tree-sitter-analyzer v3.0.0 (read, not run) against this repository's use of quarry.
No code, config, PATTERN entry or doc.go changes.

## What the report finds

- The board entry's 2026-10-18 framing is out of date.
  `go.mod` already pins quarry v0.4.0, `planglyph.EncloseAt` and `internal/failuretrace` already use `enclose`, and `lyx loom card-pack` and the kick-start pack are gone (their config keys are retired).
  Idea 2 is therefore built apart from a `lyx quarry enclose` verb, and idea 1 has no target.
- Three ideas stand, each narrower than the entry gave it: the symbol delta in review prompts, a typed reference query for the plan writer, and a doc-drift scan.
  The report sizes each and names the measurement that should come first.
- TSA's v3.0.0 work (stricter call answers, incremental indexing) leaves the "would not pay off" list unchanged.

## Design decisions

- The report states lyx's current state from the code and lists each place the entry differs, instead of keeping the entry's framing.
- Ideas are sized and caveated, with no order of work or board entries proposed, since the run is inspiration only.
- The notes the entry cites (`kickstart-pack-remove`, `enclose-checks`, `verify-diff-scope`) and the glyph audits are not re-read; the report cites them as the entry does.

## Verification

Docs only: the diff holds the report and this description, and the report's links are repository-relative backticks, so `PATTERN-markdown-link-integrity` has nothing to resolve.
