---
round: 3
focus:
  - "Pause kind in D7 and D3: which pause the pusher's reconcile writes when it stops a stopped or ended entry, and which record D3's start condition, D5's dobby-tick condition and D7's 'while the pause covers the run' read. Settle whether these name one pause and one reader. The existing pre-push check pauses through `tendercli.PausePair` (`internal/loomcli/prepush.go`), a slug-scoped entry in the hub pause record read through `pauserecord.ReadCoverage`; `lym loom pause` writes a stop condition into the run's own status."
  - "Restart of a pair's code branch in D5 and D6: when a pusher lands a pair's records side and dies before its code push, with no further commit, which mechanism pushes the code tip. D6 leaves `CodeTipMissing` unset on the owner machine while the local code branch holds the tip, the owner dobby's `settleCodeTip` (`internal/battencli/codetipsettle.go`) acts only on a `ReasonCodeTip` verdict, and D5's dobby tick starts a pusher only for a records branch ahead of origin. Settle whether the decision names exactly one restart path and whether its acceptance criterion covers a level records branch with an ahead code branch."
---

Round 3 reviews the fixes to the two open design findings: the pause D7 names, and the restart of a pair's code branch.
