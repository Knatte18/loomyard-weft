```yaml
lens: code-scope
outcome: reported
```

Task: code-scope lens. I traced each change in the diff to the goal points of the board entry.

**Decisions.** The answer key is the decision record alone. It has no `Added after Discussion` entries, and its `## Goal` has three points:
- **G1:** check whether the result of `lym gate test` shows the slot wait and the run's wall time as separate values.
- **G2:** if it does not, add them.
- **G3:** if it does, document in the verb's help where to read the run time without the wait.

**Per change:**
- **C1** — `internal/gateslot/runlog.go`: `TierSummaryLine` prints `wait %.1fs` before `wall`, and `RunRecord.SlotWaitSeconds` computes it. Traces to G2.
- **C2** — `internal/gateslot/runlog.go`: `tierSummaryPattern` and `tierDiscoveredPattern` require the `wait` field. Forced by C1, since the line's readers must still match it.
- **C3** — `internal/gatecli/report.go`: two inline wait calculations replaced by `record.SlotWaitSeconds()`. Traces to no goal point and is not forced, because the report's figures were already correct. It is the helper's second call site. It changes no behaviour.
- **C4** — `internal/gatecli/cli.go`: the help sentence "Each tier's stderr summary line shows its slot wait as wait and its run time without that wait as wall." Traces to G2 (it documents the added field) and to the spirit of G3 (where to read the run time without the wait).
- **C5** — `internal/gatecli/doc.go`: the summary-line passage now defines `wait` and `wall`. Forced by C1, since the package doc describes the line.
- **C6** — `internal/gatecli/cli.go`: the `--copies N` paragraph is removed from Long. Forced by C4, which pushed Long past its byte cap (commit a786a6b6a says so). Every fact it carried is still in the `--copies` flag help (`cli.go:132`) and the package doc (`doc.go:44`), so no finding.
- **C7** — `internal/gateslot/runlog_test.go`: `TestTierSummaryLine`. Forced by C1.

**Per decision:**
- **G1:** settled by C1. The change description says that before this change the line showed only `wall` (`_lym/landing/summary.md`, "Before this change…"), and the base's `TierSummaryLine` format string confirms it.
- **G2:** built by C1, with C2, C4, C5 and C7 following from it.
- **G3:** not applicable, since G1 found the result did not separate the values. C4 documents the field anyway.

**Findings:**

1. **NIT · scope** · `internal/gatecli/report.go:146,160`
   - **Summary:** `lym gate report`'s inline wait calculation is replaced by `RunRecord.SlotWaitSeconds`. No goal point asks for this, and C1 does not force it, because the report was already correct and its behaviour is unchanged.
   - **Evidence:** in diff `001-internal-gatecli.diff` lines 43–57, the removed `record.Start.Sub(record.WaitStart).Seconds()` is the same formula as the new helper at `internal/gateslot/runlog.go:116–122`. The summary's reason is "so the report and the summary line always agree".
   - **Fix:** none needed. Sharing one helper across two call sites is defensible, so this is recorded only as unrequested scope.

No goal point is left unbuilt.

Tool calls: none.
