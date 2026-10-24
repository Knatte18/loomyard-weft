```yaml
lens: fix-consistency
outcome: reported
```

The fix agrees with its sibling cards except in one place: card 13's ImpactSummary no longer matches its own Intent. One LOW finding.

**Finding 1 — LOW — card 13's ImpactSummary is out of date**
- Changed text: `_lyx/plan/13-testtiming-tiers.md`. The fix added `cmd/testtiming#prebuildLyx` and `cmd/testtiming#TestPrebuildLyx` to Edit. It also added this Intent sentence: "`prebuildLyx` builds nothing for an explicit `-tags tier0` either … `TestPrebuildLyx` gains a `tier0` row".
- What's wrong: the card's ImpactSummary still reads "Changes the timing tool's default and `-full` tags and its spawn scan's classification; no other caller." It doesn't mention the change to the prebuild behaviour that the card now plans.
- Evidence: the ImpactSummary line in card 13 against the Intent sentence the fix added.

Checks that found no disagreement:
- **Card 14.** The new `guardStepRun` signature and Intent agree with `scope.guards`/`computeScope` (`scope.go:143`), with `scanGuards` (`scope.go:164-167`) and with card 15's `-run` handling. No other card names `runGuardStep` except the historical `lens-review.md`.
- **Card 20.** The new `UnknownKey`, `commandRefWayForward` and helper-config text matches `commandKeyWayForward`'s new wording. It also agrees with card 19's "non-bool switch survives the move" and `Env.Resume` sentences.
- **Card 22.** The `--gate <key>` addition matches card 2's flag name (`02-gate-flag.md:19-21`).
- **Card 17.** The doc-sentence changes agree with card 17's own Intent, including that a skip carries the caller label through.
- **Card 15.** The doc rewrite agrees with the D7 Intent of the same card.

Tool calls:
1. Grepped `_lyx/plan` for names the fix touched (`runGuardStep`, `commandTestConfig`, `prebuildLyx`, `Extends`), to find twins in sibling cards I didn't have.
2. Grepped card 19 and the other cards for Command/switch and guards/prebuild mentions, since only cards 13, 14, 15, 17, 20, 22 and 31 were in my context.
3. Grepped cards 2, 7 and 10 for the `--gate` and `--guards` flag names, to check card 22's synopsis sentence.
4. Read card 18 and grepped `00-overview.md` for D6, D7 and D10, to check the card 17 doc sentences against the history-reader card and the overview.
5. Grepped `00-overview.md` for testtiming, guard step, `--open-terminal` and counts wording, because the previous grep showed no decision text.
