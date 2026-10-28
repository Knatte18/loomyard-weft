```yaml
lens: code-tests
outcome: reported
```

**Per-behaviour list (code-tests)**

- `code_reviewer_fan` becomes `CategoryNone`: `TestLoadConfig` row "a local code_reviewer_fan of \"\" is open to the local layer" (`internal/loomengine/config_test.go`) and `TestModules_EveryTemplateKeyClassified` (`internal/configreg/configreg_test.go`) pin it.
- The other five fan keys stay guards: `TestLoadConfig_Refuses`, "a local fan key is refused". The loop now lists five keys.
- The retired `darn_reviewer_fan` and `review_fan` keys are admitted: the two reworked `TestPrepareRunSettings` rows in `internal/configreg/runsettings_test.go`.
- The retired `webster_fan` and `webster_reviewer_fan` keys are renamed and admitted: the two new `TestPrepareRunSettings` rows in the same file.
- `run_settings` carries a solo `code_reviewer_fan`: the new "a solo code_reviewer_fan is admitted..." row in the same test.
- Reconcile keeps `code_reviewer_fan: ""` in the local file: the new `TestReconcileAll_SparseLayers` row in `internal/configsync/configsync_test.go`.
- The D6 profile blocks: `TestTemplateLadder`'s `solo` column, which checks presence and emptiness per block, so it also pins speed darn/pawl, solid and wide carrying no fan key; `TestTemplateDocsProfile`; the `internal/battencli/runsettings_test.go` row pins the seeded document.
- `board-scope`, `repo-invariants`, `code-affected-set` and `code-tests` prose, and the removal of `goal-scope`, `board-fit` and `code-pattern-fit`: `TestConfigTemplate_DecodesAndIsSelfConsistent`. It pins the sentences, the absence of the retired names and the fan orders.
- `twill-discussion` having 6 entries: `TestReconcile_MovesKeysAcrossModules`, `[:5]`.
- The emphasis-line exemption for `board-scope`, `way-forward` ending with the emphasis line, and the writer-covers-reviewer exemption: `emphasisLineGaps` and `writerFanGaps` run over the real template, and `TestTemplateChecks_FlagBrokenTemplates` runs them over broken maps.
- The csharp `review-code` loses `code-pattern-fit`: **no test pins it.**
  - The lens-set loop (`internal/lenses/config_test.go:512-519`) compares the loaded fan to `layer.Fans[name]`, which comes from `sets/csharp.yaml` itself.
  - The `numerics`-last check (`config_test.go:548`) does not catch a stale `code-pattern-fit` either.
  - Because `code-pattern-fit` is no longer a defined lens, a stale entry would fail `Load`/`ResolveFan` in that loop (the fan-entry-defined check, inferred, not run). So the retired lens is caught indirectly.
  - What no test pins is the exact csharp order `code-correctness`, `code-affected-set`, `code-tests`, `numerics`, which D3's acceptance criterion states.

**Findings**

1. LOW / design: `internal/lenses/config_test.go`, the `!procedureLenses["board-scope"] || procedureLenses["leanness"]` assertion, added after `procedureLenses["board-scope"] = true`. The `board-scope` half is a tautology, since the test sets that entry two statements earlier. Only the `leanness` half checks anything.
   - Fix: drop the `board-scope` half.

2. NIT / design: `config_test.go`, `TestTemplateChecks_FlagBrokenTemplates`, writer rows "a reviewer lens the writer fan lacks" and "a reviewer-only lens with no exemption". Both exercise the same gap-detected path; the second differs only in an empty versus nil `reviewerOnly` map. The "with its exemption" row is the distinct one.
   - Fix: drop the "no exemption" row.

3. NIT / design: `config_test.go`, new `code-affected-set`, `code-tests` and `board-scope` sentence assertions, plus the "Test economy" and emphasis-line negatives. They are assertions on static lens prose (rubric: a new assertion on a stencil's static prose). Some are demanded by the acceptance criteria (D1, D4); the `PATTERN.md` and "Commit messages and departures" substring checks on `code-affected-set` are the ones the criteria name least firmly.
   - Fix: keep what the criteria name and cut the rest.

4. NIT / design: the new `webster_fan` and `webster_reviewer_fan` rows in `internal/configreg/runsettings_test.go` assert the full template comment text of `code_reviewer_fan` in `wantDocs`. Any edit to that key's comment breaks them although the behaviour (rename and admit) is unchanged.
   - Fix: compare only the key and value.
   - Evidence: the rows added in the diff.

**Tool calls**
1. `grep` on `internal/lenses/config_test.go` for csharp/numerics: the manifest lacked the lens-set assertions.
2. `Read` of `internal/lenses/config_test.go` lines 496-525: to see how the expected fan is derived in the lens-set loop.
