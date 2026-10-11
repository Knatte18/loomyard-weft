# Glyphs end to end: test glyphs, traced binds, Webster kick-start pack

Plan cards now name changed test members as glyphs, every terminal pass traces the outcome of each `plan:` bind, and `begin-batch` puts the batch's resolved glyph spans into the Webster fork prompt behind one `webster.yaml` switch.
The glyph audit found that forks ignore the stencil instruction to call `lyx quarry resolve` (16 of 51 forks called quarry at all), and that bind outcomes were not countable.
The pack replaces the instruction with Go-side resolution, and the trace lines make a with/without comparison countable.

## What changes

- **Test members as glyphs.**
  `loom-template-plan.md` and `loom-rubric-plan-review.md` say a card names an existing test function, fake or helper it changes as a member glyph (`internal/x#TestFoo`).
  A plain test-file path stays only for a file the card adds tests to without changing an existing member.
  A new test function is a `**Create:**` declaration with a `plan:` handle, a new test file a `**Create:**` path.
  The rubric flags an `Edit` card that lists only a test-file path while its `Intent` changes a named test.
  No mechanical plan-gate check was added; the stencil and the rubric enforce the rule.
- **Bind trace.**
  `postBatchChecks` (shared by `record-batch` and `recover-batch`) logs at Info one line per owned `plan:` handle: batch, card, handle, `bound` with the glyph, or `failed` with one reason of `card-not-done`, `batch-not-done`, `no-delta-match`, `sibling-unmatched`, `delta-unavailable` or `language-none`.
  Passes that error before or in `DoneChecks`, in a non-`ErrQuarryUnavailable` `Delta` call or in `BindHandles` log no bind lines.
  The log adds no finding, refusal or state.
- **Kick-start pack.**
  `webster.yaml` gains `kickstart_pack` (template default `true`, toggled with `lyx config webster --set kickstart_pack=false`).
  On, `begin-batch` resolves the member glyphs under `Edit`, `Delete`, `Rename` old sides, `Move` and `Uses:` on the batch's cards and writes file, line span and signature per glyph into the fork prompt file.
  Unresolved member glyphs and every `plan:`-shaped ref are listed under an unresolved heading, never as an error.
  The fork is told to read the listed spans, not to call `lyx quarry resolve` for resolved entries, and to locate an unresolved member glyph with quarry or a grep.
  Off, the fork prompt is byte-identical to the previous render.
  `begin-batch` logs one Info line per call with the batch number, the switch state (or `language-none`) and the resolved and unresolved counts.

## Design decisions

- `BindHandles` now returns a per-handle outcome through a type `planindex` declares.
  The handle ownership rule moved from `planglyph` into the exported `planparser.OwnedHandles`, so passes that never reach `BindHandles` list the same handles.
- Resolution goes through a new `planindex.Index` method implemented in `internal/planglyph`, so `internal/websterengine` stays free of quarry and the C grammars.
- The pack is built after the dispatch re-resolution and re-baseline, immediately before `RenderForkPrompt`.
  A whole-call resolve error does not refuse `begin-batch`: every member glyph is listed unresolved and a Warn is logged.
  A `language: none` plan gets no pack.
- The pack reaches only the fork prompt file under `.lyx/webster/prompts/`.
  The `begin-batch` envelope carries no pack content or count, and Merriam's context does not grow.
- The locate sentences of step 2 moved into stencil fragments (`webster-fragment-locate-quarry`, `webster-fragment-locate-pack`), and the pack section is a third stencil (`webster-fragment-pack-section`, plus `webster-fragment-pack-empty` for a batch with no member glyphs).
  The recovery prompt always takes the quarry fragment and gets no pack.
- The default is on, so every Webster run changes its fork prompt at once.
  The comparison's off side is runs on this binary with the key `false`.
- `internal/websterengine/doc.go` documents the pack design.
  `.gitattributes` gets `eol=lf` for the four new stencils, and the `internal/websterengine` row of `cmd/lyx/testdata/test-budget.yaml` is raised.

## To check by hand

- Pack spans go stale within a batch when earlier cards edit a file, and a `no_report` re-fork reuses the first fork's prompt file; the signature fallback in the pack fragment depends on the fork following it.
- The with/without audit (grep-for-member calls and whole-file reads per fork) is not run here; it needs real loom runs after this lands, and it has no board entry once this task closes.
  Quarry-side notes `kick-start-pack` and `enclose-kickstart` wait on its result.
- `begin-batch` test row for the parse-degrade case: the plan-review finding `card8-parse-degrade-row-refused-upstream` says the case may be refused upstream of the pack build, so confirm in `internal/websterengine/beginbatch_test.go` that the row exercises the degrade path.
- `TestGofmt_ModuleFilesAreFormatted` failed once on a misaligned key in `internal/websterengine/template_test.go` and was fixed; the final full `lyx gate test ./...` and `--tags integration` runs are the plan-level verify and were not run by the forks.
