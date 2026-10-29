```yaml
lens: fix-consistency
outcome: reported
```

Fix-consistency lens: checking that the fixer's change set agrees with the sibling cards, decisions and cross-references it touches.

**LOW: the overview's card index still describes card 3's old scope.**
- Changed text: card 3's Intent now limits the release to a claim's reading: "when `journalReadLocked` fails for an id of a claim's reading … a read's reading keeps today's failure branch".
- Problem: `_lym/plan/00-overview.md`'s Card Index entry for card 3 was not updated. It still reads "a failed journal write at an acknowledgement releases the ids it left unjournaled (D3)", which covers every acknowledgement, the read verb's included.

**LOW: card 3 and card 9 now mark a read's reading in two different ways.**
- Changed text: card 3 adds "the unexported `read bool`, set by `Store.Read`".
- Problem: card 9 was left as it was and still tells a read's reading apart by its missing path. It says "a read's reading has none and logs nothing", and `daemon.awaitAck` "keeps that Warn for a read's reading alone, one with no path".
- Effect: once card 9 lands, two fields encode the same fact. Card 9 should key the Warn on card 3's `read` field, or card 3 should note that card 9's `path` takes its place.

**LOW: card 4's doc text describes hook-wait behaviour that only card 6 builds.**
- Changed text: card 4's `internal/creelengine/doc.go` text gained this Bound: "an acknowledgement whose confirmation is refused or lost shows the mail the daemon did not journal twice, once in the wake …". The claim paragraph also now names the exception "`hook-wait` once its acknowledgement is written, show the mail whatever the acknowledgement's outcome".
- Problem: hook-wait only starts writing the lines and acknowledging them in card 6 (`hookWaitExit`), and card 5 builds the client's ring acknowledgement. Through cards 4 and 5 the doc describes behaviour the tree does not have yet. Card 6's Intent edits only `internal/creelcli/doc.go`, so no later card checks these sentences against the code.
- Fix: either move both sentences to card 6, adding `internal/creelengine/doc.go` to its targets, or say in card 4 that they describe the client half that cards 5 and 6 build.

Further tool calls:
- One grep of `_lym/discussion/decision-record.md` for D1's stop-hook summary wording and of cards 4–9 for read-verb mentions. My context did not hold D1's acceptance text or the sibling cards' handling of a read's reading.
