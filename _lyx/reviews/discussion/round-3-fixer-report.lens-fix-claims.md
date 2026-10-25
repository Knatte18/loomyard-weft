```yaml
lens: fix-claims
outcome: reported
```

fix-claims lens: I checked every claim the fix added or rewrote against the code at HEAD.

Claims checked:
- "a refused judgment halts at its budget today" holds. `rejudge`'s doc at `internal/shedadapters/bouncer.go:1071` says its Stuck "never escalates, so a segment at its budget halts on it".
- "`lyx loom resume`, which re-judges the round" holds. `rejudge` archives the judge outputs, so the next Call re-judges, and the `internal/loomcli/cli.go:453` help says "A bounce-budget-exhausted halt is lifted only by "lyx loom resume"".
- "the perch runs the round's fixer on its own review before the Bouncer judges it" holds. The recipe runs Burler-Review, then Burler-Fix, which goes `on_done: Discussion-Bouncer`.
- "after a CONVERGED verdict the next round's focus file is empty" holds. `approved_focus_lists` is "an empty `focus`" (`bouncerprompt.go:75`), and when the file is missing `ensureFocus` writes an empty one (`bouncer.go:977`).
- "the decision file carries no text" holds. `circling.go:256-260` writes only the YAML header between `---` lines.
- "the round's own fixer already had every finding but the `decision` ones" holds for this segment, since the fix row holds back only `decision` findings.

Findings:

- MEDIUM: one claim in the new hard-cap sentence of D8 does not hold for every refusal.
  The changed text is "`lyx loom resume`, which re-judges the round under the recorded accept, so a judge that now rules CONTINUE or CIRCLING lets the accept pass".
  The sentence covers any held refusal, and that includes a dropped-key refusal over an accept.
  For a dropped key, a CONTINUE or CIRCLING ruling does not let the accept pass.
  Evidence: `judgmentRefusal` (`bouncer.go:1045-1063`) refuses a verdict of any kind at round 2 or later when its ledger drops a key of the previous round's ledger.
  Only the CONVERGED-specific checks (lines 1024-1043) are lifted by a non-CONVERGED ruling.
  The sentence should limit that clause to the BLOCKING-finding refusal, or add that the re-judged ledger must keep every key.

Tool calls beyond context:
- Read `ensureFocus`, because the context lacked where the next round's focus file comes from when the judge writes none.
- Grepped `loomcli` for resume behaviour at a halt, because the context lacked whether resume lifts a budget halt.
- Read the `goto` help at `cli.go:440-465`, because the context lacked confirmation that only resume lifts a bounce-budget halt.
- Grepped the circling decision writer in `circling.go`, because the context showed the header struct but not whether the file has a body.
