```yaml
lens: code-affected-set
outcome: reported
```

Lens code-affected-set: I traced what the diff's changes force outside the lines it touches. Three LOW findings came out of it: two stale or skipped sites beside the new ledger rewrite and verification file, and one rule document the new format leaves out.

**Per change, the set found and whether it is covered**

1. **New run-dir file `round-<N>-verification.md`** (`verificationPath`).
   - `bouncerverify.go` writes and reads it: covered.
   - `rejudge` now archives it: covered.
   - The parity producer (`buildBouncerVerify`) and the test fixture name it: covered.
   - The other site that archives judge outputs for a re-judge, the hard-cap halt in `settleDecisionAccept` (`bouncerdecision.go:210`), does not archive it. That is finding F2.
   - The judge spawn and render (`bouncer.go:2062`, `2157`) need no change: covered.
   - The rule document `pattern/PATTERN-sole-parsers.md` lists on-disk formats and their sole parsers, and says "A format another task adds joins this list". The new format is not added. That is finding F3.
2. **The ledger is now rewritten by Go** (`markLedgerResolved` and `renderLedger`; the yaml tags gain `omitempty`).
   - Every key the judge's ledger contract allows (key, rounds, status, class, severity, fix; `bouncer-template-judge.md:94-120`) is modelled, so the rewrite loses no key: covered.
   - `parseLedger` and its readers (soft-cap rule, carry-over, next judge, `droppedKeyRefusal`) are unaffected, since only the status changes: covered.
   - The doc of `rejudge`, a function the diff edits, still says "The judge stays the only writer of those files" (`bouncer.go:1215`), which is now false for the ledger. That is finding F1.
   - `PATTERN-tolerant-records` covers JSON records only; the ledger is YAML markdown, so it does not apply.
3. **New stencil `bouncer-template-verify`.**
   - Registry, `roleOpeningStencils` and the marker test row: covered.
   - The parent and edit directives are rendered: covered.
   - It carries no engine-specific wording, so the `ReadForEngine` bullet of `PATTERN-stencil-ownership`'s engine-block rule is not triggered. The plain `stencilstore.Read` matches the other Bouncer stencils: no finding.
   - Its claim that "each fix commit's subject names the finding ids it fixes" matches the fixer's rules (`internal/burlerengine/prompt.go:367`, `doc.go:206`): covered.
4. **New spawn, role `bouncer-verify`, on `ConfigRole` `RoleBouncerJudge`.**
   - Parity producer table entry, which the parity scan requires: covered.
   - `agentconfig` adds no role, by design.
   - `tools/tokencount/segment.go:66` `roleSuffixLabels` has no `bouncer-verify` label. It falls back to the role's own name, which is not wrong: no finding.
   - The pass is recorded as a judge pass through `recordPass(round, false, …)`. That is an attribution choice, not a broken reader: no finding.
5. **Behaviour change in `blockingEscalationDetail`.**
   - Callers `settleUndecided` (`bouncer.go:1444`) and `escalateUndecided` (`bouncerdecision.go:151`) both inherit the verification: covered.
   - Doc comments on `BouncerConfig.BlockingEscalates`, the `Bouncer` type, `settleUndecided`, `doc.go`, the shed-recipe spec, `docs/overview.md` and the three recipe comments are all updated: covered.
6. **Commit messages and departures.**
   - `Root-Cause:` on 34f6aa39e names the defect in `blockingEscalationDetail`, which the diff changes: covered.
   - No `Pattern-Departure:` or `Loosened-Check:` trailers.
   - The directive declares no PATTERN changes, so nothing to land.

**Findings**

- **F1.** Severity LOW, class consistency, at `internal/shedadapters/bouncer.go:1215`.
  - Summary: the `rejudge` doc still says "The judge stays the only writer of those files", but the diff makes Go rewrite the round's ledger in `markLedgerResolved`.
  - Evidence: `bouncer.go:1215`; `bouncerverify.go`, `markLedgerResolved`, which calls `os.WriteFile(ledgerPath(...))`.
- **F2.** Severity LOW, class consistency, at `internal/shedadapters/bouncerdecision.go:210`.
  - Summary: the hard-cap halt in `settleDecisionAccept` archives the judge outputs so a resume re-judges the round, but leaves `round-<N>-verification.md` beside the re-judged ledger.
  - `rejudge` now retires that file because "that ruled on the retired ledger's keys", and the same reasoning applies here.
  - Today the round's escalation record keeps `closeVerifiedBlocking` from reusing the file.
  - Evidence: `bouncerdecision.go:210`; `bouncer.go:1220`; `bouncerverify.go`, `closeVerifiedBlocking` and `verification`.
- **F3.** Severity LOW, class consistency, at `pattern/PATTERN-sole-parsers.md:6-19`.
  - Summary: the diff adds an on-disk format, the verification file, whose sole parser is `parseVerification`, but does not add it to the list, which says "A format another task adds joins this list with its sole parser".
  - The Bouncer's existing ledger, verdict and focus formats are also absent from the list, which softens this.
  - Evidence: the sole-parsers background as read earlier in this context (its "Formats and their sole parsers" section); `bouncerverify.go`, `parseVerification`.

**Tool calls**

1. grep for `judgeOutputs` and `verificationPath` callers, "only writer" claims and bouncer-judge role tables: the manifest lacked the other archive sites and role-keyed tables.
2. `bouncer.go:2052-2075` and `2148-2175`, `bouncerdecision.go:200-212`, `tokencount/segment.go:50-85`: to read the archive and role-map sites the grep found.
3. Judge stencil lines 88-130 and fixer stencil commit wording: to check that the ledger rewrite keeps every contract key, and to test the verify stencil's commit-subject claim.
4. grep across `contracts` for the commit-subject convention: the first search found no match in the burler stencils.
5. grep of `burler-step-fix.md` and `burlerengine` for the commit-subject rule: to locate where the fixer is told that a subject names its finding ids.
