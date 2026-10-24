```yaml
lens: leanness
outcome: reported
```

Task: leanness review of the decision record and support log, looking for content to move or cut.

**Findings**

1. MEDIUM, Decision 2: rejected alternatives sit inside a Decision.
   - The rationale says "the board names three candidate maps … needs no coverage run and no index lyx lacks (quarry answers declarations, not references)".
   - The decision body also says "No card names its tests under `Uses:` for this".
   - Both argue against options that were not chosen. The spec's compaction rule is "Never list a rejected alternative here", and the support log's Rejected alternatives already holds both (coverage, quarry references, `Uses:`).
   - Fix: keep a one-line rationale (narrows inside a shared test project; misses are bounded by the whole gates) and drop the rest.

2. MEDIUM, Decision 6: the closing rationale clause "a lyx-kept restore fingerprint would add a record and a stale-`obj/` failure mode for a saving of seconds" is a rejected alternative.
   - The support log already holds it as "A lyx-kept restore fingerprint".
   - Fix: keep the no-op-restore fact the decision rests on and cut the comparison.

3. MEDIUM, Decision 9 says again what earlier sections already say.
   - Its first and third bullets point back to Decisions 1–5 and add nothing new.
   - Only the second bullet carries something the plan writer needs: there is no new record for the verified tree, because the green record already covers the slow tier. That bullet is also pinned by an acceptance criterion.
   - Fix: turn that one fact into a Constraint line, or one sentence in Decision 5's area, and delete Decision 9.

4. LOW, Decision 11 repeats the Scope bullet for `docs/dotnet-hub.md` and lists keys that Decisions 4–8 and 10 already define.
   - The plan writer needs the page's existence and its contents, and both are already stated.
   - Fix: fold it into the Scope bullet and delete the Decision.

5. LOW, Decision 7 is mostly explanation.
   - The binding content is that no environment sets any of the three disabling variables, and that the docs name the server opt-in. That fits in one Constraint line.
   - The reparenting and `runproc.End` story is deliberation. It also makes an unverified claim about existing code: no runproc or proc code was read to confirm that nodes escape the tree. That claim belongs to another lens.
   - Fix: shrink it to the rule and move the rest to the support log.

6. LOW, Auto-mode assumptions restates Decisions 2, 6, 8 and 10 and points at rejected options ("coverage-based … were not chosen (support log)").
   - Only the last bullet (linked compile items fall back to whole) is an assumption that isn't already a decision, and it duplicates Decision 3's fallback list.
   - Fix: keep the bullets that state a real assumption without a decision behind it, and drop the restatements.

7. LOW, Scope's last out-of-scope bullet ("A lyx-managed restore step and any lyx change to MSBuild node reuse … (Decisions 6 and 7 say why)") repeats Decisions 6 and 7.
   - Fix: cut it.

8. LOW, Notes for the plan writer: the last bullet (the `cs` sandbox hub's retired list-shaped `gates` "is no input here") states something irrelevant to the plan.
   - Fix: cut it.

9. LOW, Acceptance criteria: the last criterion ("Existing gate behaviour under `go` and `none` … unchanged") duplicates the last Constraint word for word in substance.
   - Fix: keep one of the two. The acceptance criterion is the better home, since it names its test evidence.

10. LOW, Decision 10 is borderline. It writes out all three Models rule texts. Under the completeness-before-leanness test they stay, because the plan writer never reads the board entry and needs the text for the docs page. Not a cut.

No further tool calls were made: my context already held both files, the spec and the rubric.
