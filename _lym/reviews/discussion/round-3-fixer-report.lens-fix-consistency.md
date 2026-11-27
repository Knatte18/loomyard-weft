```yaml
lens: fix-consistency
outcome: reported
```

fix-consistency: whether the round-3 changes to D3, D7, D10 and their criteria agree with the parts of the decision record they touch.

1. **LOW: D8's dedup criterion was not updated to match the new D7 criterion.**
   - Changed text: D7's new criterion line, "that a second failed-state hand-back at the row gets a new key and a second severe post when its verdict is `operator`".
   - The D8 tier0 criterion in `internal/slackengine` still names only "a distinct second halt of the slug within that window …, a fold into the same history entry among them" as the case that mentions again. It does not name a second failed-state halt with no history entry.
   - That is the case F37 says D8's dedup used to mute. Its second post now depends on the new key reaching D8's dedup, and D8's list of example halts does not include it.

2. **LOW: D3 and D7 describe the stop record's contents differently.**
   - Changed text: D7's "the stop record names the open stop's halt entry, when it has one, and the child read reads the four from that entry, or from the stop record when it names none", and "records them on the driver's stop record (D3)".
   - D3 defines the stop record as the place the driver "keeps" the stop identity. In D3, the history index of the halt entry is carried by the settle records, not by the stop record.
   - D7 adds two things D3 never states: the stop record names the open stop's halt entry, and it holds the four facts. D3's definition, which D7 cites as "(D3)", now describes less than D7 relies on.

No further tool calls.
