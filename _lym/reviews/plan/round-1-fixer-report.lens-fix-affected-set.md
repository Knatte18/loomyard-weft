```yaml
lens: fix-affected-set
outcome: reported
```

Task: re-run the set searches behind the set-premise findings (F4, F5, F6, F10) and check that the fix covers every member.

Unchecked: F1 (its set, the callers of `battencli.Cancel`, is new code with only `fronttask.Remove` as a planned caller, so there was nothing to search), F8 and F9 (single sites, not sets).

F5, F6 and F10 re-searched and covered:
- **F5:** `lym adopt check` appears in `gateslot/doc.go:108`, `PATTERN-test-speed.md:37`, `adoptcli/doc.go:73,88` and `check.go:1`. Card 5 covers all of them through its Docs paragraph, its `adoptcli/doc.go` Edit and its `check.go` header line.
- **F6:** the headers of `adopt.go`, `check.go` and `deps.go` are covered. No other adoptcli header goes stale.
- **F10:** the only callers of `tierNotice`, `suiteMessage` and `checkMarkOf` are in `check.go:151,198` and `adopt.go:338`, inside `runCheck` and `adopt`, which card 5 already edits.

F4: members not covered by the fix:

1. LOW — card 11's F4 Intent covers `beamcli/doc.go` only.
   The same sentence is in the `beam create` long help at `internal/beamcli/cli.go:499`: "Dobby never starts a beam project, so only this verb, a hand pass or "lym adopt" starts or resumes its run."
   That help goes stale once adopt resumes nothing.
2. LOW — `internal/battencli/owner.go:102` (`OwnedElsewhereError`'s refusal) is outside every card.
   It still says "run `lym adopt %s` from this machine's prime, which takes it over and resumes its run".
   After card 11, adopt holds the run and the green check resumes it.
3. LOW — `runBeamSteps`' doc comment at `internal/adoptcli/adopt.go:627` ("the adopt that takes the parent over and so resumes the landing here") is outside card 11's Edit list and Intent.
   Card 11 rewords only the `heldBeamWayForward` text that this comment mirrors.
4. LOW — card 11 lists `adoptcli/doc.go`, but its Intent rewrites only steps 4 to 6.
   Two lines elsewhere in that doc still say adopt resumes:
   - `adoptcli/doc.go:63`: "`lym adopt <beam>` on the parent's owner, or `lym adopt <parent>` on the beam's owner, resumes it"
   - `adoptcli/doc.go:66`: "…`lym adopt root` for the prime, which resumes the landing here"

Calls:
- grep across internal/docs/pattern for texts saying adopt resumes: reran F4's search, which the context lacked.
- grep for `lym adopt check` across the tree: reran F5's search.
- adoptcli file headers plus callers of the re-signed functions: reran the F6 and F10 searches.
- sed of `adopt.go:626-632`, `beamcli/cli.go:494-500` and `battencli/owner.go:96-100`: identified the symbols holding the uncovered texts.
