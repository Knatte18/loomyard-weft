# Rewrite the comments and docs of the repository to the scribe skills

This change rewrites every comment, `doc.go` and markdown doc in the repository to follow `scribe:prose`, `scribe:golang-comments` and `scribe:code-quality`.
It changes comments and prose only, never code.
The aim is comments that state the present contract and the why, stand alone, and break lines the way a lint can check.

## What changed

- Go comments: file headers, package docs, doc comments and inline comments, in `cmd`, `internal` and `tools`.
- Test comments: `_test.go` files were rewritten from comment digests, each comment edited after a read of the span around it.
- `doc.go` files and markdown: package docs, `contracts` specs and stencils, `docs`, `pattern`, `crucible/agents`, `plugins/ly` and `tools/sandbox`.
- The common rewrites:
  - fixed-column wrapping became one sentence per line, each line under 120 characters of text;
  - history, task narrative and finding ids were replaced by the present state;
  - a doc comment that leaned on another named symbol now states what its own symbol does and why;
  - comments that restated the code, or pinned a count or a list of current callers, were cut or reworded;
  - comments that named a symbol, file or verb that no longer exists were made true.
- Raddle is live: a recipe, `internal/raddleparser`, the `_lym/raddle/` run directory and `lym report raddle` all exist.
  Only the squash-merge-back of `_lym/raddle/` was dropped, and comments say so.
- `lym loom lint-comments --base 23518a94` is clean over the whole change.

## Constraints that held

- Every edit is a whole `//` comment line or markdown prose.
  No string, identifier, import, directive line or end-of-line comment was changed.
  The one added blank line separates the `tools/wordswap/swap.go` file header from `package main`, so the package keeps one package doc.
- In `contracts/stencils`, no `{{...}}` marker, engine block marker, heading, table column or instruction meaning changed, and a stencil changed only where it was false.
- Left out as records: `docs/research`, `docs/benchmarks` and `docs/quarry-holistic-fix-log.md`.
  Generated files and `testdata` are also untouched.
- Strings and raw-string help text are out of scope, so none was edited.

## Not fixed

- Code and help-text matters the workers found stay unfixed, since this change edits no code and no string.
  Among them: stale headings in the `configengine` doc, `lymcwd.ResolveWithAnchor` with no non-test caller, and stencil paths under `{{.specs_dir}}/loom/` that name no existing folder.
- Comment defects the workers reported but left are not fixed either: claims kept as written because they could not be confirmed against the code, and boundary fragments where two workers' spans met.
- Comment lines of 120 or more characters outside the spans a worker edited remain.
- Several files were edited by more than one worker at once, among them tests in `cmd/lym`, `internal/adoptcli` and `internal/websterengine/render.go`; a reviewer should read those diffs whole.

## History

The first pass of this cleanup is kept in commit `1d4deacc` and reverted by `b45ef8ba`.
This change is the skill-led pass that replaced it.
`git diff --stat 23518a94` lists the files it touches.
