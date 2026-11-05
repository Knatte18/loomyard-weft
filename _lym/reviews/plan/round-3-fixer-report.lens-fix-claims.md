```yaml
lens: fix-claims
outcome: reported
```

fix-claims lens: checking that every claim the fix added or rewrote holds against the code at HEAD.

**[MEDIUM] Card 15's new Bound sentence says containment reads only four inputs, but it also reads the run's status row from the exempt run directory**

- **Changed text:** `_lym/plan/15-drift-judge-seat.md`, the new `Bound:` sentence after the exempt-paths paragraph: "...but decides no containment, which reads only the records, the sealed expectation, the baseline and the current capture."
- **Evidence:**
  - Card 8 (`08-drift-seal-baseline-checks.md:54,58`) has the check use `PastRow(d.Rows, status row, expectRow)`, with "The status row is read through `Deps.Status`".
  - Card 10 (`10-drift-containment-check.md:51`) skips the whole check when the current row is not past Drift-Expect.
  - `Deps.Status` is wired in `internal/loomcli/driftwiring.go:113` and reads the run's `status.json`.
  - `internal/shedrun/paths.go:122` puts `status.json` in the run directory (`_lym/shed/&lt;segment&gt;`), which is the exempt `RunDirRel`.
  - So containment does read a file under an exempt path, and the "reads only" list is false.
- **Possible fix:** the bound can still hold, but the sentence has to say why. Card 15:68 says loom commits `status.json` at every row transition, so a later row's check reads loom's rewritten status row and never one the judge left. The sentence should name the status row as an input and give that reason, not claim containment never reads it.

The other claims added or rewritten by the fix hold:
- Card 15: the exempt paths hold no code, expectation, `scope.json` or `judged.json`, because `scope.json` sits in driftengine's own durable directory (`internal/driftengine/paths.go:55`). The stop report reads friction notes and archives as halt notes (`internal/reportengine/stops.go:97-115`).
- Card 12: the new row's "no agent, no seal commit" matches the Intent's ordering.
- Card 16:
  - The interim row's prefix claim matches the card's line-127 prefix rule.
  - The two constants' values match the route strings.
  - Card 8 does create `Deps.Rows`, so the route derivation has a source.
- Card 17: the replacement sentence matches card 17's Intent.
- Cards 5 and 6: the new rows are refusals the cards' own Intents prescribe.

Tool calls:
1. Grepped `shedrun` and `loomengine` for the run, friction and drift record directories, to check the "hold neither scope.json nor judged.json" claim. My context did not have these path constructors.
2. Grepped cards 8 and 10, `shedrun/paths.go` and loomcli's drift wiring for what containment reads from status and seed, to check the "reads only" claim. My context did not have card 8, card 10 or the `Status` wiring.
