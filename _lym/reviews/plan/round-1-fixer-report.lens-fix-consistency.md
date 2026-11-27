```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fix's changes agree with their sibling cards, decisions and cross-references (fix-consistency lens).

**MEDIUM: card 1's claim about card 6, and card 6's walk over `neve-qdbus`**
- Changed text, `01-kwin-probes.md` Intent: "card 6's `TestRecordings_EveryRecordingReads` reads every recording it takes".
- Card 6 has no reader for the `neve-qdbus/*.txt` answers. Card 8's `recordedExec` reads them, and card 6's Uses do not list them.
- Card 6's test "walks every directory under `testdata/recordings` and runs the reader its files call for over it, failing on any unexpected error".
- Before the fix, `neve-qdbus` came with the runner card, after the readers card. Now card 1 commits it before card 6, so card 6's own walk meets a directory none of its readers handles.
- So the claim is false for `neve-qdbus`, and card 6's walk has no rule for that directory. The fix needs one of two changes: card 6's walk skips the qdbus answers or names them, or card 1's sentence names card 8 as their reader.

**LOW: card 13 names two `window.go` files without saying which is which**
- Changed text, `13-reed-window-placement.md`: "`window.go` changes whole: its file comment, `openTerminalWindow` and `windowCmd`". This one is `internal/reedcli/window.go`.
- A later, unchanged line in the same Intent says "into `window.go`'s package comment", meaning `internal/desktophook/window.go`.
- The card's Edit list holds both files, so a reader cannot tell which `window.go` each sentence means. Spelling each as a full path settles it.

I checked these and found them consistent: the overview's probes-before-code decision against card 1; card 6's placeholder and `ApplyReport` tiles against card 1's scripts; card 10's `CurrentOrder` and `Prime` against card 11's `reportUpdateFrom` and step 5; card 9's `ReportUpdate` against `reportUpdateFrom`; the `within` bounds in card 8 and card 11; card 12's references to card 3 and card 8; the renumbered Applies-to lines.

No further tool calls made.
