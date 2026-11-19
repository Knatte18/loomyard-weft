```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the change set agrees with its sibling cards, decisions and cross-references (fix-consistency lens).

1. MEDIUM. Card 11, `LiveRed`: "An absent report is red once the round's fix row ran, which the round's `harden-commits-&lt;round&gt;.json` (card 9) shows". The commits file doesn't actually mean the fix row ran its gates. Card 9 step 4 writes it on "a fix row whose call returned no error". The burler fix producer returns no error when "the highest complete round is still unjudged, it hands control on without spawning" (`internal/shedadapters/burlerfix.go` Call doc, lines 137–142). So a fix row that spawned no fixer, and ran no `live` gate, still leaves a commits file. `LiveRed` then answers true, and the Bouncer escalates instead of auto-accepting a round whose live gate never had a chance to run. Either card 9 should write the commits file only when the fix half ran, or card 11 needs a signal that ties to the gate actually running.

2. LOW. Card 11 changes `RoundFacts` ("`RoundFacts` names such an absent report as red for the same reason") but leaves its siblings as they were:
   - The same card still says "A file the round lacks is named as absent, never an error". That doesn't contradict the new rule, but it doesn't mention the red case either.
   - `TestCampaignRoundFacts` gains no row for an absent live report beside a commits file being rendered red. Only `TestCampaignLiveRed` covers the new case.

3. LOW. Card 6 step 2 records a refused copies call "in the live report as refused, with its refusal text". But step 4's list of what the report holds still reads "per run, passed, the started count, the failing lines of its output and, for the copies run, the clamp line", with no refused field. Card 11's `RoundFacts` list ("the live report's mark, runs and started counts") likewise never renders the refused state, so the judge's facts can't tell a refused copies run from a green one.

No further tool calls made.
