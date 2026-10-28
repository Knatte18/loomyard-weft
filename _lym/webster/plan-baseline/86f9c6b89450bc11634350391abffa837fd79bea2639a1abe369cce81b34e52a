# Card 3 — hold-notice-creel

**Edit:**
- `internal/shuttleengine#holdNotice`
- `internal/shuttleengine#holdNoticeSettle`
- `internal/shuttleengine#TestWait_HeldTurnEndNotice`
- `internal/shuttleengine#TestAttach_LegacyAskingAndHeldRuns`
- `internal/shuttleengine/doc.go`

**Uses:**
- `internal/shuttleengine#MessageTail`

**Intent:**
The hold notice names creel instead of `SendMessage` (D3).
`holdNotice`'s way-forward sentence becomes: "When its marked question is in your creel mail, answer it with lym creel post --reply <id> \"<text>\"; otherwise post to it with lym creel post <strand> \"<text>\", ending your message with %q.", with `<strand>` the strand name and `%q` `MessageTail`, so the line still names the strand and the tail and holds no newline.
The rest of the notice, its delimiters, bounds and last-message part, is unchanged.
The comments that describe it follow: `holdNotice`'s doc comment, `holdNoticeSettle`'s comment ("long enough for a SendMessage to reach an idle peer" becomes long enough for creel mail to reach an idle peer) and the hold-notice paragraph of `internal/shuttleengine/doc.go`.

`TestWait_HeldTurnEndNotice`'s expected line is rewritten to the new sentence and its strand-guid row's `"SendMessage to strand-1,"` check becomes `"lym creel post strand-1 "`; `TestAttach_LegacyAskingAndHeldRuns`'s `"SendMessage to "+strandName+","` check becomes `"lym creel post "+strandName+" "`.
Both rows already reach the notice, so they fail if the old wording returns.

**ImpactSummary:** One notice line's way-forward wording and its comments; the two tests that assert the line change with it, nothing else reads it.
