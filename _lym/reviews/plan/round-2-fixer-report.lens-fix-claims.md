```yaml
lens: fix-claims
outcome: reported
```

fix-claims lens: I checked every claim the Card 11 Intent rewrite makes about `internal/loomcli/doc.go` against the code at HEAD.

no findings

- "loomcli doc.go enumerates them in two sentences" is true. Line 471 and line 577 are the only places that list the escalation causes. The other hits mention circling or the hard cap without listing causes: line 104 names the `circling` and `decision` groups, line 292 a recorded circling decision, line 330 the perch's hard cap and line 521 a pending circling decision.
- The first quote, "a pending escalation of cause `hard-cap`, `circling` or `decision`", matches the end of line 471. Line 467 introduces that list as "A stop is eligible only if the verb's closed list admits it", so "the stops the verb's closed list admits" is accurate.
- The second quote, "asks the arbiter at an awaiting stop with a pending hard-cap, circling or decision escalation only", matches line 577 word for word. It sits in the paragraph that line 570 opens with "It decides the stop by a closed rule table…", so describing it as the table's sentence holds.
- The sentence "Its doc comment and arbiterstop's doc.go list the admitted causes with `stalemate`" only regrouped the old wording. It makes no new claim about the code.

Tool calls:
- 1 grep and sed on `internal/loomcli/doc.go`: my context held only the two lines the grep matched. I needed the surrounding sentences to check how the card describes each one, and a wider search for cause lists that do not mention `hard-cap`.
