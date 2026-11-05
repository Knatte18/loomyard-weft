```yaml
lens: code-affected-set
outcome: reported
```

**Task:** the code-affected-set lens. I took each moved or changed path or file shape and looked for every caller, doc, test and comment that still points at it.

**Per change: the set found, and whether it is covered**
- **Study folders moved under topic folders.**
  - Go comments and templates: covered. `internal/batcher/doc.go:29`, `internal/lenses/config_test.go:334`, `internal/lenses/template.yaml:39`, `internal/shuttlecli/smoke_guardrail_test.go:2`, `internal/shuttleengine/claudeengine/startup.go:5` and the three codexengine files all carry new paths.
  - `README.md` rows 141–146, `docs/overview.md`, `docs/benchmarks/*` and `crucible/review-prompt-template.md`: covered. `README.md:127` links the `docs/research/` directory, which still exists.
  - `tools/tapestryrename/tables.go:111` and its test use the `docs/research/` prefix, which is still valid.
  - A tree-wide grep for `docs/research/<old-name>` and `research/<old-name>` outside `.tsv` files and verbatim captures finds one survivor, listed below.
  - Rule documents: nothing under `PATTERN.md` or `pattern/` names a moved path.
  - `docs/quarry-holistic-fix-log.md:5` cites `docs/research/quarry-port-log.md`. That file is absent before and after this diff, so it is not caused by the diff.
- **README-only studies became `report.md`.**
  - A search for basenames present at the base commit and gone at HEAD finds five: `twill-plan-bench.md`, `twill-plan-bench-sonnet.md`, `twill-plan-bench-sonnet-medium-fable.md`, `twill-discussion-bench.md` and `plan-split-tune-2026-11-02.md`.
  - The twill plan-bench and plan-split-tune reports were rewritten to the new names; the `twill-discussion-bench.md` reference in `deviations.md` was missed.
  - No remaining prose points at a renamed `README.md`.
- **Design notes moved to `docs/`.** Covered. Their relative links were fixed in `reed-exploration.md`, `reed-proposal.md`, `psmux-tui-behavior.md` and `reed-hooks-exploration.md`. `overview.md` lists all five moved notes.
- **Scripts' root lookup replaced by `git rev-parse`.** Covered. All 20 sites are converted. The staged-copy flow in `turn-savings-measure-2026-11-04/run-all.sh` and `review-turn-audit-2026-11-01/run-all.sh` no longer needs a `docs/research` mirror. Sibling `../turn-audit-2026-10-28` lookups stay valid because the siblings share the `turns/` folder.
- **`docsLinkIsResearchData` predicate.** Covered. The doc comments and the subtest are updated, and `vocabScanAllowlist` gains entries for the two moved notes that name weft and warp.
- **`PATTERN.md` declared changes.** None are declared and none apply. No `Pattern-Departure:` trailer is present.
- **Commit message claims.** Borne out. README-only reports became `report.md`, the benchmarks, crucible template and `overview.md` were updated, and `docs/research/README.md` indexes every study.

**Findings**

1. **LOW / consistency** — `docs/research/twill/twill-discussion-bench/deviations.md:1` (line 5 also names it) says "the full record is `twill-discussion-bench.md`".
   - That file is now `report.md` in the same folder, so the reference is stale.
   - The decision record demands that no old reference survives.
   - Evidence: the five-name basename search above, and `git ls-files docs/research/twill`.

2. **LOW / consistency** — `docs/research/glyph/glyph-audit-2026-10-15/COMMANDS.md:13` still copies from `.../loomyard/docs/research/glyph-audit-2026-10-13/{extract2.jq,...}`.
   - Line 101 of the same file was updated to `docs/research/glyph/glyph-audit-2026-10-13/`.
   - The old path in the step-0 reproduction command no longer exists, so the file is inconsistent with itself.
   - It is an absolute path in the older hub checkout, so the fix is to point it at the new folder or state that the checkout path is historical.

**Considered and not raised:** old paths inside verbatim captures such as `calls.tsv` and `findings.tsv` (for example `turn-audit-2026-10-28/calls.tsv`, `concrete-review-findings-2026-11-03/data/findings.tsv`). Editing them would alter measurement data, so they are not findings.

**Tool calls (1):** `git grep -n -E 'docs/research' -- . ':(exclude)docs/research' ':(exclude)*.tsv'`, to list every reference to the directory from outside it, which my context lacked in one view.
