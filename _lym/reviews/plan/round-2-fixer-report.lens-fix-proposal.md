```yaml
lens: fix-proposal
outcome: reported
```

Lens: fix-proposal. I challenged the proposals carried out as given: F14 (card 5 wiring) and F16 (doc.go moved to Edit on cards 14 and 16). F15 was fixed differently, so it is outside this lens.

**F14: no finding.**
I tried to break it with the case where an existing tier1 create test now reaches the real pick on the developer's machine and writes the real account file. That case does not hold. The claude accounts live in the machine layer `&lt;Board&gt;/.lym/config/&lt;module&gt;.machine.yaml` (internal/configengine/doc.go:54-55). A `hubforge` hub has its own Bolt, so the layer holds no accounts and the pick is a no-op, as the card says.

**F16: two findings.**

- **MEDIUM.** Card 16's new sentence "The re-send paragraph … says such a first turn end is neither counted nor re-sent" misses a sentence its own behaviour falsifies.
  - That paragraph also says "A run with no retries waits such a turn end out to its deadline" (internal/shuttleengine/doc.go:194).
  - Card 16 says "a spec without re-sends starts fresh too" (line 29) and has a row: "a copied resume on a spec without `APIErrorRetries` starts fresh".
  - An implementer following the named rewrite would leave doc.go:194 false for a copied resume. Card 14's re-send rewrite has the same gap for a usage-limited turn end the switch handles on a spec with no retries.
- **LOW.** Cards 14 and 16 name the doc.go hold paragraph but not its twin in `loadSkillsThenPrompt`'s doc comment.
  - That comment says "A load turn whose provider API fails skips nothing: loadSkillTurn holds it, and the prompt waits until a load turn ends" (internal/shuttleengine/run.go, the comment above `loadSkillsThenPrompt`).
  - The same switched and refused-resume load-turn cases falsify it.
  - Both cards list `Run.loadSkillsThenPrompt` under Edit, so the comment can be changed, but no card names that sentence.

Tool calls:
1. grep of `fabricengine.ConfigDirs` and cards 2 and 4: to learn where the machine layer and the account file live, which my context lacked.
2. grep of configengine for the machine layer: same question, not answered by the first call.
3. sed of internal/configengine/doc.go lines 50-64 and 126-140: to read the machine layer's path.
