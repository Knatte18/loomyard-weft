```yaml
lens: leanness
outcome: reported
```

Task: a leanness review of the decision record and support log, looking for content to move or cut.

**Findings**

1. MEDIUM — Notes for the plan writer, first and third bullets.
   They give caller lists and the burlerfix notifier coupling, which the writer's Step 2 fence excludes as plan-level detail ("MUST NOT gather exact signatures... dependency lists").
   The `burlerfix.go` coupling does carry a constraint: D8 needs the exclusion without the notice.
   D8 already says that, so the bullet repeats it.
   Fix: cut both bullets, or cut the caller list and keep one short hint.
   `relevant.md` already lists `decision.go`, `orchengine/prompt.go` and `burlerfix.go`.

2. MEDIUM — Notes for the plan writer, fourth bullet (the list of stencils and specs that name parent review).
   This is call-site enumeration, which the rubric's "Do not flag" says belongs to Plan.
   D2 already names the stencils that are removed.
   Fix: cut it, or move the paths into `relevant.md`.

3. LOW — D2's last paragraph, the run waiting on a parent-review round at upgrade.
   It repeats the Constraints bullet "D2 keeps Discussion-Write's name, so only D11 restarts parked runs", and an acceptance criterion repeats it again.
   Fix: keep the D2 paragraph and drop the Constraints bullet's restatement, keeping only its point about row positions.

4. LOW — Two Constraints bullets restate D8.
   - `PATTERN-shed-recipe-registry` ("D8's switch is a Bouncer row config key") repeats what D8 already says ("by a row config key").
   - `PATTERN-friction-capture` claims the existing causes write a friction note, but nobody checked that. It may be a premise without evidence, so it belongs with whichever lens checks premises.
   Fix: merge each into its own decision, or keep only the bare PATTERN id.

5. LOW — Auto-mode assumptions.
   Each bullet restates a pick that the Decisions and the Question ledger both record already: three copies of each pick.
   The section is required, so a one-line pointer per pick is enough.
   Fix: shorten each to "D1: PR-Gate premise false; agent goes whole".

6. LOW — D1's last rationale sentence ("The board entry's 'keep it at pr-gate' names a row that no longer exists...").
   It is said again in Auto-mode assumptions and in support-log Turn 2.
   Fix: keep the fact in D1's rationale and cut the copies.

7. LOW — D8's second bullet spells out the escalation brief's contents and the full sequence of verbs.
   That is close to stencil wording, but the attack-surface rule needs the way forward stated, so it stays by the completeness test.
   Fix: none needed; noted only.

8. LOW — Open risks, third bullet ("review-discussion grows by two forks... both rounds on this hub's discussion_reviewer_fan").
   This is a cost note tied to one hub's config, not a risk the plan acts on.
   Fix: move it to the support log or to D5's rationale.

I made no further tool calls.
