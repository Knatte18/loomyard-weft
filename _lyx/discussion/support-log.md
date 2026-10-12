## Interview

Autonomous session: no operator answered, so every turn below is the writer's own exploration and pick.

1. Read the board entry `webster-pack-read-set` and the related entries `kickstart-pack-measure`, `kick-start-pack`, `enclose-kickstart` and `weaver`.
   The board's Want fixes four changes: bench wording, span-only FRESH-READ, per-card packs, one-line `var`/`const`; the operator ruled out inlining source and ruled in reading beyond the pack.
2. Read the pack code (`planglyph/pack.go`, `planindex` pack types, `websterengine/render.go`, `beginbatch.go`'s `buildKickstartPack`) and the webster stencils, and the bench card `07-e1-pack.md` in the quarry worktree.
3. Checked quarry v0.2.1: `Symbol.Kind` distinguishes `var` and `const`; `Signature` is verbatim and untruncated; grouped specs carry the spec's own text, so a first line names the member.
4. Settled per-card grouping with one resolve call (D1, D2), the bench wording per card (D3), FRESH-READ split by pack mode to keep the pack-off baseline byte-identical (D4), truncation in `planglyph` (D5) and unchanged log counts (D6).
5. Lens review: six forks, all returned.
   - goal-scope MEDIUM, step 1 reads the card file in its own turn, so the per-card read splits into two turns: fixed, step 1 becomes a pack-mode marker (D3).
   - goal-scope LOW, the measurement has no owner: fixed, Scope Out says queuing it is the operator's call.
   - goal-scope LOW, a null result's meaning is unanswered: rejected, the board fixes the Master out, and the measurement's verdict is the operator's.
   - board-fit MEDIUM, a per-card `Pack` ties the type to cards against `enclose-kickstart`: fixed, `Pack` stays card-agnostic behind a per-card wrapper (D1).
   - board-fit MEDIUM, old pack-on counts are not a baseline: fixed in Scope Out.
   - board-fit LOW, same type concern for the Burler side: fixed by the same D1 change.
   - board-fit LOW, `weaver` unnamed as a consumer: rejected, no dependency either way.
   - pattern-fit MEDIUM, `PATTERN-leaf-packages` does not cover `planindex`: fixed, the constraint cites `planindex`'s own contract.
   - pattern-fit LOW, fragments need registry rows: fixed in Scope and Notes.
   - pattern-fit LOW, extend the existing pre-pack bytes test: fixed in Notes.
   - pattern-fit LOW, producer-pointer and no-denied-recovery compliant: no action.
   - scenarios MEDIUM, later cards meet shifted spans: fixed, Open risks and an acceptance criterion on the signature rule.
   - scenarios LOW and premises LOW (merged), handles-only card ambiguous: fixed in D2 and the criterion.
   - scenarios LOW, empty per-card unresolved list: fixed, left out (D2).
   - scenarios LOW, language-none byte identity: fixed in the criterion.
   - scenarios LOW, heading must use the pointer display rule: fixed in D2.
   - premises LOW, grouped signature carries the keyword: fixed in D5's rationale.
   - premises LOW, stencil version skew from a new required marker: rejected, the body and fragments ship together through the existing stencil lifecycle, which this task does not change.
   - leanness MEDIUM, Auto-mode assumptions repeat decisions: fixed, cut to the one assumption.
   - leanness LOW, byte identity stated three times: fixed, dropped from Constraints.
   - leanness LOW, Open risks restate D4 and the Goal: fixed, replaced by the shifted-span risk.
   - leanness LOW, Goal lists mechanisms: fixed.
   - leanness LOW, generic constraints: fixed, cut.
   - leanness LOW, "pack reaches the prompt file only" repeats today: fixed, cut.
   - leanness LOW, symbol names duplicated in Scope and Notes: fixed, Scope names packages only.

## Rejected alternatives

- Inline source spans in the prompt: the operator ruled it out.
- Make the pack the fork's read set with a tool-call budget (the `kick-start-pack` note's reviewer rule): the operator wants the fork free to read more when the card needs it.
- Reword FRESH-READ for every mode: changes the pack-off prompt and breaks comparability with the pack-off runs already counted in `kickstart-pack-measure`.
- One quarry resolve call per card: multiplies quarry cost by the card count for no change in answers.
- List a shared glyph once, under its first card: the fork at the later card would not find its spans under that card.
- Truncate every signature to its first line: loses parameters of a multi-line function signature, which the board did not ask for.
- Truncate in the renderer from a kind field on `PackDeclaration`: adds a field `planindex` needs only for this; `planglyph` already sees quarry's kind.
- Replace `Pack` with per-card groups outright: ties the shared type to cards, while `enclose-kickstart` and the Burler side need a card-less pack.
- Keep step 1's separate card-file read and put the span read at the locate step: makes the card file and its spans two turns.
- Place each card's spans inside the card-pointer list: mixes the always-present pointer list with the switchable pack and changes the pack-off prompt.

## Review rounds

_No rounds yet._

## Question ledger

- Q1: Should a glyph two cards name be listed under both? Auto-pick: yes (D1). Resolved.
- Q2: Should FRESH-READ change for pack-off and recovery prompts too? Auto-pick: no; pack-on only, via a marker (D4). Resolved.
- Q3: Does the one-line rule cover functions with multi-line signatures? Auto-pick: no; `var` and `const` only, per the board (D5). Resolved.
- Q4: Where does the `var`/`const` truncation live? Auto-pick: `planglyph`, when it files quarry's answer (D5). Resolved.
- Q5: Do the log line's counts change to per-card? Auto-pick: no; distinct refs per batch (D6). Resolved.
- Q6: Does the fork get a read budget? Auto-pick: no; the operator wants it free to read more (Scope). Resolved.
- Q7: How does the per-card parallel read fit step 1's card-file read? Auto-pick: step 1 becomes a pack-mode marker (D3). Resolved.
- Q8: Does `Pack` itself become per-card? Auto-pick: no; a per-card wrapper around a card-agnostic `Pack` (D1). Resolved.
- Q9: Does a card with nothing unresolved show `none`? Auto-pick: no; the list is left out (D2). Resolved.
