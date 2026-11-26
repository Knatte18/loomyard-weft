```yaml
lens: code-affected-set
outcome: reported
```

I reviewed the code-affected-set lens: everything outside the changed lines that reads or describes the reuse rule, the `range_key` field and the halt wording. One BLOCKING and three LOW findings.

**Per change, the set found:**

- **`JudgedFile.RangeKey` / `range_key` in `judged.json`:**
  - Written by `newRulings`, which `storeRulings` (`verify.go:1147`) and `storeCaseRulings` (`verify.go:1328`) call. Both are covered.
  - Read only by the reuse check in `listFiles`. Covered.
  - Written to disk through `internal/state` (`recordsguard.go:146-200`, `doc.go:37`), and decoded with plain `json.Unmarshal`, which ignores unknown keys (`recordsguard.go:71-80`).
  - An older binary that drops `range_key` from an uncaused ruling only causes one extra judge run. This agrees with `PATTERN-tolerant-records`; no finding.
  - Test and fixture readers are `verify_test.go` and `verdict_test.go`; none needs the key. Covered.
- **Reuse rule in `listFiles`:**
  - Callers: `checkAndJudge` (`verify.go:304`) and `passCommand` (`verify.go:1240`). Both cover it.
  - The `pass` early return at `verify.go:168` fires only after a pass, so an uncaused halt never reaches it. Covered.
  - `checkAndJudge`'s doc comment (`verify.go:257`) is left stale. Finding 3.
  - The judge's stencil is left out of step. Finding 1.
- **`uncausedReason` and its third way forward:**
  - Asserted by `verify_test.go:1293-1294`, which still matches, and by the new row at `:1355`. Covered.
  - Shared by Drift-Recheck through the same engine. Covered.
  - `docs/overview.md:1122` and `:1371` are not updated. Finding 2.
  - `orch-role.md:274-286` names no ways forward, so it is unaffected.
  - The wording claims more than the key delivers. Finding 4.
- **Package doc (`doc.go`):** consistent with the code apart from finding 4.
- **Commit messages:**
  - `e6393683f` carries a `Root-Cause:` trailer, and it names the defect the diff removes.
  - The range has no `Pattern-Departure:` or `Loosened-Check:` trailer, and none is needed: no test assertion was widened.
  - No PATTERN changes are declared, so none need to land.

**Findings**

1. **BLOCKING, design.** Location: `contracts/stencils/drift/drift-template-judge.md`, Step 2, the file rules.
   - The advertised third way forward, "commit the dropped checks in a test", cannot clear the halt, because the judge's instructions still force an `uncaused` ruling.
   - The stencil rules a file `none` only when "the diff widens or removes no assertion, tolerance or limit in a test". It rules a check `caused` only when "a `Loosened-Check:` trailer in `commit_messages` names that check", and `uncaused` otherwise.
   - A deleted harness whose checks moved elsewhere still removes assertions, and the moving commit carries no trailer. So the re-judge the diff now triggers must, per its instructions, rule the deleted file `loosened`/`uncaused` again.
   - The new test passes only because its fake judge returns `fileRulings = nil` (`verify_test.go:1177`).
   - The diff changes no stencil wording to let the judge count checks carried into another listed file. So the halt text's promise (`loosened.go:191`) is untrue for a real judge, which works against `PATTERN-refusal-way-forward`.
   - Evidence: `drift-template-judge.md`, Step 2, the bullets "`none` when…" and "`caused` when a `Loosened-Check:` trailer…"; `loosened.go:191`; `verify_test.go:1173-1179`.
2. **LOW, consistency.** Location: `docs/overview.md:1122`.
   - The line still reads "An uncaused one makes the row Stuck until a `Loosened-Check` trailer commit or a revert, then `lym loom resume`".
   - It omits the new third way forward, and it omits that a new commit re-judges an uncaused ruling.
   - `:1371` ("an uncaused one makes it Stuck the same way") inherits the gap for Drift-Recheck.
   - Evidence: `docs/overview.md:1122`, `:1371`.
3. **LOW, consistency.** Location: `internal/driftengine/verify.go:257`.
   - `checkAndJudge`'s doc still says "A stored ruling that a loosened check has no measured cause is a Stuck, with no judge".
   - Such a ruling is now reused only while the range key is unchanged; once the range changes, it goes back to the judge.
   - Evidence: `verify.go:254-257`; `loosened.go:107-109`.
4. **LOW, consistency.** Location: `internal/driftengine/loosened.go:185` and `:191`, `internal/driftengine/records.go:176`, and the claim in `_lym/landing/summary.md` "What changes".
   - The texts say any new commit, or "a later commit", has the judge rule again. The halt says resume "has the judge rule on the range's files again".
   - The range key hashes only the file diffs and trailer lines (`loosened.go:49-57`). A commit that leaves both unchanged, such as a message-only commit with no trailer or a change and its revert, does not re-judge.
   - Only uncaused or changed files are re-listed, not "the range's files".
   - Evidence: `loosened.go:49-57`, `:107-109`, `:185`, `:191`; `records.go:176`; `summary.md` line 13.

**Tool calls**

- 1: `docs/overview.md:1116-1124` and `:1366-1373`, `orch-role.md:274-286`, and a repo-wide grep for the old way-forward wording. The context had only grep hits for these files, not their text.
- 2: `verify.go:248-258` (`checkAndJudge` doc), `decodeRecord`, and the `PATTERN-tolerant-records` background, to judge the `range_key` rewrite path. None of these was in context.
- 3: grep of `recordsguard.go` and the `driftengine` imports for `internal/state` use, to confirm how `judged.json` is rewritten. The write path was not in context.
