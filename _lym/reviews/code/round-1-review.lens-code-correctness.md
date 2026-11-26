```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens. I traced every changed function and branch with concrete inputs.

**List, per function or branch: the case I ran and the result**

- **`roundCheck.carriedFinding`, `HardenerRound` true, carry moved to an unlisted file** (`roundcheck.go:278`).
  - Input: class unchanged, severity MEDIUM→MEDIUM, path `a.go`→`b.go`, `HasChanges` false.
  - The class and severity checks run first and pass. The paths differ. The `c.ctx.HardenerRound ||` short-circuit returns nil before the no-changes-file refusal.
  - Correct for the stated intent. A raised severity is still refused by the earlier check, so that order is right.
- **`carriedFinding`, `HardenerRound` false.**
  - Input: path moved, `Changed` = [`c.go`].
  - It falls through to `namedByAmendment` and then to the refusal, as before. Correct.
- **`roundCheck.newFindings`, `HardenerRound` true.**
  - Input: `Incremental` true, which `roundcontext.go:218` sets for a non-amendment generation, so this branch is reachable. New MEDIUM finding at `z.go:1`, no origin.
  - `parentCovered` is false, origin is not parent, then `continue`. It passes.
  - Correct. The uncovered `origin: parent` refusal still runs before the exemption, which is right.
- **`newFindings`, `HardenerRound` true, `origin: webster` finding with `HaltReason` set.**
  - The early `continue` skips the `websterOutside` counter, so two webster findings outside the changes file both pass.
  - Harmless: a hardener row never sets `HaltReason`, because only the plan review row wires `HaltConflictDir` (`entries_burler.go:185`). Correct.
- **`newFindings`, non-incremental round (amendment).**
  - It returns nil first, so the result does not change. Correct.
- **`priorEntry` held-finding rules in a hardener round.**
  - These are untouched by the exemption: a held finding must still be resolved with a quoted title or stay open. Correct, and it matches the claim that the class, severity and parent rules still hold.
- **`checkRoundInRunDirectory`** (`validate.go:137`): rename only.
  - Input: `BouncerRunSubdir` returns an error. It returns the parse-alone note. Unchanged and correct.
  - Input: not found. `HardenerRound` is false. Correct.
- **`BuildRoundContext` / `RoundContextSources.HardenerRound`, and `NewBurlerReviewProducer` / `roundProfile`.**
  - The field passes straight through. On a builder error, `roundProfile` logs and reviews in full, as before. Correct.
- **`decisionRuleMarker(round, checkpoint, firstRaisedGates)`.** Every case below is correct.
  - (1, 2, false): round-1 rule, no CIRCLING.
  - (1, 2, true): identical to (1, 2, false).
  - (2, 3, true): round-1 rule. No lighter clause and no parse-error fallback, which is not needed because the rule is already round 1's.
  - (5, 2, true): round-1 rule plus the CIRCLING bullet. CIRCLING reads only the facts' earlier-open list, which Go still renders whatever the flag.
  - (5, 2, false): lighter rule plus CIRCLING, as before.
- **`Bouncer.RenderJudgePrompt`**: it passes `b.cfg.FirstRaisedGates`. The zero value false keeps every non-hardener Bouncer on the old rule. Correct.
- **`bouncerEntry` harden block** (`entries_bouncer.go:372`): `FirstRaisedGates` is set only inside `if hardenRound`, so no other segment gets it. Correct.
- **Go settle paths outside the prompt** (`settleUndecided`, `softCapRule`, `designWrongKeys`):
  - None of them reads whether a key was first raised, so no Go path disagrees with the new flag.
  - Soft-cap auto-accept still counts every open BLOCKING or MEDIUM non-mechanical entry, whatever its class. That is unchanged and needs no edit here.
- **`TestBouncer_JudgePromptGatesAFirstRaisedKeyWithFirstRaisedGates`.**
  - Round 1 runs with the writing fake. Round 2 swaps in a non-writing fake, and `CallOK` only requires a nil error.
  - It reads `GotSpec.Prompt` from the round-2 fake, so the asserted prompt is round 2's. Correct.

**Findings**

None. No branch I ran produced a wrong result, a swallowed or misreported error, or an inconsistent state on a failure path.

**Tool calls**

1. `grep` for `Incremental` in `internal/shedadapters/roundcontext.go` and for `CallOK` in the shedfake kit. My context lacked whether a hardener round is incremental, which decides whether the new `newFindings` branch is reachable, and what `CallOK` asserts, which decides whether the round-2 test call can pass with a non-writing fake.
