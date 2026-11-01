# Add lym's design invariants as PATTERN entries

Six code invariants lived only in the orch's own notes, so no writer, perch or judge read them, while every one of them reads `PATTERN.md` in full.
Each is now an entry in `PATTERN.md`, and two carry a background file.
This change adds documentation only; no Go code changes.

## What changes

A new `## Code` section in `PATTERN.md` holds three entries:

- `PATTERN-doc-go-documentation`: a package's `doc.go` documents the package; a rule other code must follow is a PATTERN entry, never a `doc.go` paragraph.
- `PATTERN-named-constants`: a name or path that can go stale is one named constant, declared once and referenced everywhere.
- `PATTERN-no-hard-coded-choice`: lym hard-codes no language, engine or branch name.
  Background in `pattern/PATTERN-no-hard-coded-choice.md`.

Three entries join existing sections:

- `PATTERN-fabric-isolation` in the GitHub and pairing section, after `PATTERN-github-auth`.
  Background in `pattern/PATTERN-fabric-isolation.md`.
- `PATTERN-shed-shared-machinery` in `## Shed and loom`: what every phase machine needs is built once in shed's packages.
- `PATTERN-lym-instructions-in-code` in `## Agents and prompts`: lym's own instructions to agents live in its code, stencils and specs, never in heddle.

## Design decisions

- Only the entries whose one line cannot carry the rule get a background file: fabric isolation and the no-hard-coded-choice rule.
  The other four fit in the overview line.
- Both background files treat an existing violation as debt: a change that touches it moves the knowledge out and never extends it.
  The entries do not require a sweep of existing code.
- None of the new entries is marked `(test)`; no test enforces them.

## To check by hand

- That the six entries fit the overview's headroom and that the repository's PATTERN checks (entry format, background links) pass; none was run for this change.
- That the fabric background's definition of fabric as `internal/fabricengine` and `internal/fabriccli` matches the packages you consider fabric.
