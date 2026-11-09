```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals that the fixer carried out as proposed (fix-proposal lens).

**1. MEDIUM — F3 as carried out: `envelopeOf` collecting over `Subjects` can meet a subject with no outcome**
- **Changed text:** card 7, step 5: "rendered as a list in the order `collect` gives, over the record's `Subjects` without `root`". Card 6: "`envelopeOf` (card 7) names the `Subjects` without `root` to `collect`".
- **Summary:** card 6 seeds outcomes only into `runSteps`' in-memory run outcome. `envelopeOf` builds its slugs only from the merged parts. A record that stops before `stepPairs` writes any per-slug field has no outcome for a named subject. Two cases are a refused reconcile, which F2 now routes through `StoppedPart`, and the standing-move refusal, which runs no step. `collect` then dereferences a nil pointer.
- **Evidence:** `adopt.go:1037-1038` runs `all = append(all, *outcomes[slug])` for every named slug. Today's reconcile refusal returns `finish()` with no `result.Slugs` (`adopt.go:254-265`), so its envelope carries no slugs at all.
- **What to settle:** `envelopeOf` must either name only the subjects the merged parts hold, or say it seeds them. Seeding would list `{slug: alpha}` items on a reconcile refusal, which today's envelope never carries.

**2. MEDIUM — F5(b): the stopped end loses the only assertion that `runAdopt` names the stop's way forward**
- **Changed text:** card 8, `TestRunAdopt`: "each row asserts only the code and that the envelope or the error reaches the writer, since `TestFollow` pins the interrupted text".
- **Summary:** the review's grounds cover only `interrupted`, whose error text `follow` builds and `TestFollow` pins. For `stopped`, `follow` returns the record. `runAdopt` itself renders `output.ErrFields` naming the record's `WayForward` (card 8 step 3). That way forward is a field beside `Envelope` (card 3's `Record`), not inside it.
- **Effect:** "the envelope reaches the writer" no longer pins the D1 stop output. No other test asserts it.
- **Repair:** the stopped row should keep asserting the way forward.

**3. LOW — F6: the new sentence on `heldBeamWayForward`'s doc contradicts card 6's own step 6**
- **Changed text:** card 6 Intent: "`runBeamSteps`' and `heldBeamWayForward`'s docs, since the landing resumes by hand with `lym tender resume` and `lym loom resume`".
- **Summary:** step 6 of the same card says the text names `lym tender resume <slug>` only when `paused` is set, "and otherwise `lym loom resume` in its pair alone, since no entry stands for the tender resume to lift".
- **Concrete case:** an adopted beam with no own-scope entry and no moved run. Under the sentence as written, the doc would name a tender resume that has nothing to lift.

**4. LOW — F4: the reworded `TestRunWorker` heading misdescribes its remaining rows**
- **Changed text:** card 7: "`TestRunWorker`, one row per D2 criterion the `TestAdopt` rows do not reach".
- **Summary:** these rows remain and are not D2 criteria:
  - "An invalid request (`--keep-local` with `--drop-local`) is refused before the lock and any step" is D1's criterion: "one with invalid arguments is refused before any step".
  - The standing-move row ("A record done through `owner`, re-run while a move record stands…") restates no D2 criterion.
- **Repair:** the heading should name D1 too, or say "per criterion".

**5. LOW — F2 as carried out: `StoppedPart`'s fate on the standing-move refusal is undefined**
- **Changed text:** card 6: "A later run of that step that ends without error clears `StoppedPart`, and a later stop replaces it."
- **Concrete case:** a record stopped at `deploy`, so its `StoppedPart` holds that deploy outcome. It is re-run while a move record stands. `runSteps`' refusal "ends the record `stopped` at `from`… and no step runs".
- **Gap:** that later stop has no part. The text doesn't say whether it replaces `StoppedPart` with an empty part or leaves the earlier deploy outcome to be merged into this stop's envelope. Card 7's standing-move row would see either.

No further tool calls made.
