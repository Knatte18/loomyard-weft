```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fix set agrees with the sibling cards, rows and cross-references it touches (fix-consistency lens).

No findings.

- **Card 14 (F1):** no other card, and not the overview, describes how `integrateWeft` builds its commits. The only places that mention `RunWithEnv`, the temp index or the plumbing are card 14's own Create line and Intent. The claim that the plumbing "stays in `fabricengine`" agrees with `resetWeftToIntegrated` staying in `destroy.go` and with `PATTERN-fabric-git`.
- **Card 8 (F2):** it now says "card 6's `internal/fabricengine/anchormarker.go`", which agrees with card 6:42 creating that file as new. Card 6 comes before card 8, so the file exists when card 8 needs it.
- **Card 17 (F3):** the four fresh-run errors in the new `TestChangeAnchor` row (pass wait, parent lock, dobby end, log-sink close) match the Intent's fresh-run list one for one. The new sink-reopen row matches the Intent's standing-record list ("an error reopening the sink after `Apply`").
- **Card 17 (F4):** the new adopt clause agrees with card 11:36. There, `stoppedBranches` stops an owned-elsewhere slug over `anchor_stopped` the same way as over a diverged branch, and the existing way forward names `--keep-local` and `--drop-local` (`internal/adoptcli/adopt.go:356`). `sharedStopText` renders a prime-weft `anchor_stopped` with the `--set` as its way forward. The owner-refusal rows in `TestChangeAnchor` assert only the `--set` text, so none of them goes stale.
- **Card 13 (F5):** the caller list now agrees with card 15:61 (`Close`), card 15:66 (`Apply` failure), card 15:70 (`Finish`) and card 18:40 (create).

Tool calls:
1. Grepped the plan for the temp-index, `lyxtemp` and `RunWithEnv` mentions and for `anchormarker.go`, to find twins of the F1 and F2 text in cards my context did not hold.
2. Read the rest of card 14 and the overview's package mentions, to check for an impact or package list that would need `gitexec`. My context did not hold those lines.
