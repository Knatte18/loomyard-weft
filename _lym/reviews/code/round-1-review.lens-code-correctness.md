```yaml
lens: code-correctness
outcome: reported
```

Task: lens code-correctness, running the changed functions and branches on concrete inputs.

**Per function or branch, case run, result**

- `isJudgementPath`, `rel == judgementRel`: `_lym/drift/judgement.yaml` returns true. Correct.
- `isJudgementPath`, archived sibling: `_lym/drift/judgement-20261106T083132Z.yaml` has the same dir and `.yaml` extension. The trimmed base `judgement-20261106T083132Z` matches the archive pattern, with segment `judgement` equal to the trimmed base of `judgementRel`. Returns true. Correct.
- `isJudgementPath`, collision suffix: `judgement-20261106T083132Z-1.yaml`. The greedy `(.+)` backtracks to `judgement`, so the segment is `judgement`. Returns true. Correct.
- `isJudgementPath`, a different name: `judgement-notes.yaml` doesn't match the stamp pattern, so it returns false. Correct.
- `isJudgementPath`, a different extension: `...Z.txt` fails the extension check, so it returns false. Correct.
- `isJudgementPath`, a different directory: `_lym/drift/old/judgement-<stamp>.yaml` fails `path.Dir`, so it returns false. Correct.
- `isJudgementPath`, a double stamp: `judgement-<stamp>-<stamp>.yaml` gets segment `judgement-<stamp>`, which is not equal, so it returns false. Lym's rotation never produces this name.
- `isJudgementPath`, judgement path with no extension: both extensions are empty and the logic still holds.
- `isJudgementPath`, an anchored path such as `src/x/_lym/drift/judgement.yaml`: `path` is slash-based, and `Changes` already converts `judgementRel` with `filepath.ToSlash`, and git paths are slash-separated, so it works on Windows too. Correct.
- `Changes`, code checkout: the exemption is now the predicate, and it was an equality check before. This is consistent with how `judgement.yaml` was already exempt on both checkouts. Correct.
- `Changes`, records checkout: `recordsExempt` is the predicate OR `isLoomRecordsPath`. Correct.
- `loomRecordsDirs`, `AnchorRel "."`: `filepath.Join(".", "_lym/usage")` is `_lym/usage`. Correct.
- `loomRecordsDirs`, `AnchorRel "src/backend"`: gives `src/backend/_lym/usage`. Correct.
- `isLoomRecordsPath`, usage entry: `_lym/usage/session-x.json` matches by the `entry+"/"` prefix. A sibling like `_lym/usage-x/a` or `_lym/usagefoo` is not exempt, because the entry has no `-` suffix and needs an exact match or the `/` prefix. Correct.
- `isLoomRecordsPath`, friction entries: the `friction` and `friction-` entries behave as before. Correct.

**Findings:** none.

**Tool calls:** none. My context already held every definition I needed.
