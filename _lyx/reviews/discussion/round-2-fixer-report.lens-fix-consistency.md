```yaml
lens: fix-consistency
outcome: reported
```

Lens fix-consistency: I checked that the changed text still agrees with its sibling decisions, criteria and cross-references.

1. MEDIUM. **Decision 3's empty-test-scope paragraph and the empty-test-scope acceptance criterion now read as one build per opted-in tier.**
   The new text says "a tier that opts in with `shared_build` runs the build step (Decision 6) and no test command".
   The criterion says it "runs, for each listed tier that opts in with `shared_build`, the build step and no test command, inside that tier's slots".
   Decision 6's first bullet runs the build "once ... inside the slot hold of the first opted-in tier in plan order", and the build criterion says "runs once per call".
   With two opted-in tiers in an empty test scope, the new per-tier wording has the build run in each tier's slots, which contradicts Decision 6.
   The per-tier rule should say that an opted-in tier runs no test command and that the build runs once, in the first such tier.

2. LOW. **`support-log.md` line 51 is the stale twin of the F6 fix.**
   It still says "interactions F5 (exit code after build failure): fixed; the build is the first failing run."
   Decision 6 and its criterion now use the plan-order first-failed-run rule.

3. LOW. **Decision 2's result bullet does not cover a card whose Targets are all inert.**
   The F1 fix's Decision 3 card bullet says a Target whose file is inert "is dropped from the seed", so a card whose every Target is inert has an empty seed.
   Decision 2's result bullet tells the empty scopes apart only "apart from an empty or inert-only diff".
   Decision 3's empty-scope paragraphs name only an empty diff and an inert-only diff.
   No sibling states what an all-inert card runs: no tier but a `carry: false` one, as an inert-only diff does, or something else.

Tool calls:
- One grep of `support-log.md` and `decision-record.md` for webster, whole-gate, nearest-ancestor, untracked and exit-code wording. My context did not hold `support-log.md` beyond the Q2 line, so I needed it to find twins of the changed text.
