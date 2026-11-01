```yaml
lens: way-forward
outcome: reported
```

Lens: way-forward. I checked whether the next step each card's refusals, Stucks and recovery steps name can work from the state they leave behind.

**MEDIUM — D8/D7, cards 5, 6, 7, 14: the part check rewrites a part's `plan:` handles, so a join refusal on that export can never be cleared**
- Running `validate-plan --part` puts a part's plan through `resolvePass`, and that calls `CanonicalizeHandles` (`internal/planglyph/handle.go:207-227`). It rewrites every declared `plan:` handle to `plan:<expected-glyph>` across the part's card files on disk.
- When quarry's canonical spelling differs from the writer's draft, the exporting part's cards end up with the canonical handle. `parts.md` and the dependent parts still use the draft spelling.
- Card 6's `JoinParts` then refuses "export not created", because it matches handles exactly.
- The way forward it names (the writer fixes the part, then the chair runs `lym loom join-parts` again) cannot work: every recheck canonicalizes the handle again.
- The dependent part's draft reference passes its own part check through `PartScope.Handles`. In the joined plan it would dangle.
- No card states that exports must be canonical, compares them after canonicalization, or rewrites the parts file and dependent parts. The writer stencil (card 14 Step 3) tells the writer only to spell an export "exactly as the parts file spells it".

**LOW — D6, card 11: the claim that the writer error cases are retried is only half true**
- The card says "The two error cases keep the run's transient classification retrying them."
- `shedtransient` retries `ErrNotStarted` always (`internal/shedtransient/shedtransient.go:30`). It retries `ErrAgentDied` only when `spawnRefusedRecently` (`:32`).
- A writer that dies otherwise fails the run with an error that names the writer seat and no next step. That matches today's chair mapping, but the card's statement is wrong and no way forward is named.

**LOW — card 7 `ValidatePart`: the missing-parts-file refusal goes to a session that cannot act on it**
- The missing-parts-file error names `lym loom goto --to Parts-Write`. It reaches a part writer, or the chair, through `validate-plan --part`.
- A run agent does not run goto, and neither the writer nor the chair stencil (card 14) says what to do with this refusal, such as reporting it to the chair or the parent.

**LOW — card 3 `Validate`: the `record-missing` finding names a step the Parts-Write agent cannot use**
- Its way forward is `lym loom validate-discussion`. The agent cannot write the record, so the gate spends its three re-prompts and goes Stuck.
- A wiring-level error, or a way forward addressed to the parent, would end it sooner.

**LOW — card 19 `joinPartsVerb`: no message is set for an absent status**
- Step 1 lumps "the status is absent" with the halted and other-row cases but gives two messages only: halted at the row, and at another row.
- No message or way forward is chosen for an absent status, which `TestJoinPartsVerb` expects to refuse.

**LOW — card 14 `PartsPlanTable`: a failed table build leaves live strands behind**
- On a resume with a live chair, a missing or malformed parts file fails the table build before `Probe` runs. The live chair and writers are never stopped.
- The named way forward, `lym loom goto --to Parts-Write`, then leaves `parts-plan*` strands holding their names. The next `Parts-Plan-Write` start refuses in `checkName` with `lym reed remove`, a step the refusal never mentions.

**LOW — card 15 `NewPartsPlanRotator`: a retry after a half-finished preparation loses the prior-plan block**
- If the plan rotation succeeds and the parts archive then fails, the plan is already rotated.
- On the retry the plan rotation is a no-op and returns no amendment, so the new chair never gets the prior-plan block.

**LOW — card 1 `LoadTapestryConfig`: one refusal names no way forward**
- The `parts_max` below 1 refusal names no next step, unlike the `parts_chair_timeout_min` refusal.

**LOW — card 6 `JoinParts`: a write failure mid-join leaves a half-written plan directory**
- The plan directory's card files and `00-overview.md` are removed before the joined files are written. A write failure in between leaves it half-written.
- The card names no way forward. Running `join-parts` again would clear it, but the chair stencil's "never run join-parts again once you have written into the plan directory" may read as forbidding that.

Tool calls:
- grep of `internal/planglyph/handle.go` (`CanonicalizeHandles` doc) and `internal/shedtransient/shedtransient.go` (`ErrAgentDied`/`ErrNotStarted`): my context did not have what `CanonicalizeHandles` rewrites or how these errors are classified.
