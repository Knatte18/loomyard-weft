```yaml
lens: leanness
outcome: reported
```

Task: a leanness-lens review of the decision record and support log. I looked for content to move or cut, and noted anything else on the way.

**MEDIUM — D2: the list of roles is a perishable enumeration.**
- D2 names every role that commits code: "the Webster implementer, the darn writer, the sampler session, the Code-Burler fixer, the gate fixer, the merge fixer and the PR-Review agent".
- The rule itself already defines the class: "every role that commits code in a recipe with a Code-Review segment". Listing today's members goes stale when a role is added or renamed.
- The rubric's do-not-flag list counts call-site enumeration as Plan-Write's job.
- Keep the class definition and the shared-fragment rule, and cut the list. The plan writer loses nothing, because the class tells it what to find.

**LOW — Scope "In" repeats Decisions.**
- The four "In" bullets restate each decision's files and mechanisms (rubric, trailer, judge request, verb, config key, `on_reroute`, friction note). D1 to D7 say all of this again.
- Shrink "In" to one line per board idea, or cut it and let "Out" carry the scope boundary, which is the part only Scope holds.

**LOW — Auto-mode assumptions repeat D4 and D5.**
- "The default for `pr_review_small_max` is 3" is stated in D5.
- "The PR findings file uses burler's existing review format" is stated in D4.
- Only the first bullet, which findings sets count, adds anything, and D4 already says that too ("the parent's review and a list of findings the operator gives").
- Cut the two duplicates. The question ledger already records these as auto-picks.

**LOW — D3 repeats a Constraint.**
- "Go reads both through fabric, never by running git itself" repeats the `PATTERN-fabric-boundary` / `PATTERN-gitrepo-client-boundary` Constraint line. Keep it in one place.

**LOW — Some Constraints are generic PATTERN restatements.**
- These entries apply to any change of this kind and carry nothing specific to this design, so the plan writer gets them from PATTERN anyway:
  - `PATTERN-cli-cobra`: "the verb is a command in the `lym loom` subtree"
  - `PATTERN-shuttle-provider-seam`: "stopped through the stop verb", which D4 already says
  - `PATTERN-sole-parsers`, which D4 already states
- The other lines tie an entry to a specific decision and earn their place.

**LOW — The second Open risk repeats D3's bound.**
- "The judge's reading of 'widened' is judgment…" restates D3's "The judge decides what counts as a test".
- Either fold the refactor false-positive into D3's Bound, or cut it.

**LOW — D6 bundles two mechanisms.**
- "The miss is recorded" (the friction note) and the re-entered round reading the PR findings as review input are separate mechanisms.
- The review-input half belongs with D4's send-back path, or in its own decision.
- This is emphasis only. Both halves carry requirements, so nothing should be cut.

**MEDIUM (not a leanness finding) — D3: the judge may spawn on nearly every drift run.**
- The trigger is "files that exist at the run's baseline commit and that the committed code changes", which includes non-test source files.
- Almost every task modifies an existing file, so the judge would spawn on nearly every drift-enabled run, even with no changed cell and no test touched.
- The record states no bound on this cost.
- Options:
  - Narrow the trigger to files the judge or Go can tell are tests.
  - State the cost as an accepted bound.
- This is in the bound-breaker or walkthrough lens's territory, so noted briefly.

**Not flagged:** the Goal restating the board entry is fitting for a Goal, and the support log's Turn 2 summary of picks sits where deliberation narrative belongs.

No further tool calls were made.
