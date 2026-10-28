```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked pairs of decisions in the review-setup-tuning decision record for one undoing or contradicting another.

**Findings**

- **MEDIUM** — `## PATTERN changes` against D7 and D3. The `PATTERN-config-layers — changes:` bullet repeats the entry's current overview text, but D7 changes only the background file. The declared change therefore never shows in a `PATTERN.md` diff, and two checks will fire on it:
  - The plan gate's `pattern-line-unprescribed` check (`internal/loomshed/patternplan.go`) reports a declared changed entry whose bullet the worktree overview still holds as the base does, unless a card fences the line. That forces a no-op `PATTERN.md` card.
  - Code-Review's "Declared PATTERN changes landed" check expects the diff to carry each declared change with its text. D3 hands that check to `code-affected-set`, so a reviewer may raise a false BLOCKING finding.

  The "Notes for the plan writer" line ("the overview line keeps its text") confirms the bullet declares nothing. Fix: drop the bullet and keep the background edit in D7 and Constraints. The alternative is a real overview-line change.
- **LOW** — D3's rationale says "the rubric bullets it applied move with it so no rubric check loses its lens". The `code-pattern-fit` prose also named "A broken invariant" and "A pattern the parent changed", and D3 carries neither. No stencil under `contracts/` or `internal/` holds those two bullets, so nothing is lost. The claim should still say those names were dropped as dead references, so the plan writer does not restore them.

**Pairs checked**

- D5 and D6 (same key, opened then set in profile blocks): held. Profile blocks pass `checkLocalDocument`'s `ValidateLocal` once the key is `CategoryNone`. `""` decodes as `FanList{""}`, and `ValidateFanKeys` skips empty entries.
- D5 and D6 with the five other fan keys still guards: held. D6 sets only `code_reviewer_fan`.
- D5's bound and `code_max_bounces`: held. The stronger off-switch already opens to the same layers.
- D1 and D2 against the writer-covers-reviewer check: held. `board-scope` is in both discussion fans, `repo-invariants` is the one named reviewer-only lens, and the `affected-set` → `forced-edits` absorption is untouched.
- D1 and the dormant `discussion` fan: held. It is renamed in the same decision.
- D2 and `twill-plan`: held. `way-forward` stays defined and stays in the writer fan.
- D3 and D4 (two code lenses over the rubric): held. "Test economy" goes to the lead only, and D3's added bullets don't overlap D4's narrowed scope.
- D4 and D6 (a solo reviewer in `light`): held. The solo reviewer applies the whole rubric, "Test economy" included.
- D3 and the `csharp` layer: held. The layer fan is changed in the same decision.
- D6 and the hub's seed-only `profiles.yaml`: held. Open risks says it.
- D7 and D5 (background Guard text and the reclassification): held. Background-only, but see the MEDIUM finding.
- The acceptance criteria for D1–D7 each map to their decision. The D5 criterion checks both the opened key and the five keys still closed: held.

**Tool calls**

- Grepped `internal` for the declared-PATTERN-change checks. My context had no code reading `## PATTERN changes`.
- Read the header of `internal/loomshed/patternplan.go` to see when `pattern-line-unprescribed` fires. I needed it to judge the unchanged-text bullet.
