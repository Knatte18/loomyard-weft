```yaml
lens: leanness
outcome: reported
```

Leanness lens: find what to move out of or cut from the decision record.

**Findings**

1. **MEDIUM — Goal paragraphs 2–3 and Scope "Out" bullet 1.** The landed work is described twice.
   - The Goal names `internal/runfsm`, the tracked intents, the own-session resume and `pairteardown.EndSubjectProcesses`.
   - Scope Out lists the same items again: states, act rule, `Judge`, `Aside`, intents, `RegisterOrch` and the subject-process end.
   - Fix: keep the list once, in Scope Out. Cut the Goal down to its first paragraph plus the "What is left" sentence.

2. **MEDIUM — D8 Rationale.** "mapping the code's empty slug would touch every owner reader D1 to D6 rewrite, for a display fix" argues against an alternative.
   - The support log's Rejected alternatives already carries it ("Mapping the empty parent slug to `root` in `hubgeom.ParentOf`").
   - The requirement itself already sits in D8 bullet 3 ("keep returning the empty slug").
   - The spec's compaction rule puts rejected alternatives in the support log only. Fix: cut the clause and keep the first rationale sentence.

3. **LOW — D1 bullet 2.** "Today each commits and spawns a detached push (`internal/tendercli/runintents.go`, `internal/battencli/cancelcmd.go`, the pull-request record's committer, and the status commit that carries the fixer counts)" lists today's call sites.
   - The rubric marks call-site enumeration as Plan's job.
   - The requirement ("push through the same synchronous path") stands without the list. Fix: move the paths to `relevant.md`.

4. **LOW — D2 bullet 1, the last bullet of D4 and the last sentence of D5's upgrade bullet.** Each restates which function has a production caller ("its production caller in the rejected-push path…", "The catch-up's entry point has its production caller at the dobby start…", "This first read is the hold setter's production caller").
   - The `PATTERN-test-only-code` constraint repeats all three.
   - Say it once. Fix: keep them in Constraints. In D4, keep only the requirement "adopt calls it too (D6)" if it is needed for the D6 link; D6 step 4 already states it.

5. **LOW — D5 bullet 5, second sentence.** "every verb the act rule guards refuses except on the fix task and a landing past its parent-side merge" repeats D5 bullet 2.
   - Bullet 2 already covers the hub-wide `wait`, the fix task excluded from `Held`, and the landing exception.
   - Only the "orch is up" clause is new. Fix: keep that clause and cut the rest.

6. **LOW — Auto-mode assumptions.** The section restates picks the decisions already state outright: root in D8, the stop lifting its entry and going on past an unconfirmed idle in D9, the verb names in D9 and D10.
   - These are legitimately recorded as assumptions.
   - Fix: shorten each bullet to a pointer. The support log ledger Q1–Q8 already carries the reasons.

7. **LOW — Notes for the plan writer, bullet 3.** "`hubforge` builds hub fixtures" is test-plan detail the plan writer finds itself, and it is generic. Cut it, or keep the two-machine fixture hint alone.

8. **LOW (emphasis) — D9 Rationale, second sentence.** The decision needs it, since it carries the fact that hub-wide resume restarts nothing. The support log's Q2 repeats it. It belongs in the record, so keep it there; no cut.

9. **LOW (emphasis) — Notes, bullet 1 (the code dependencies).** It is the split guidance raddle needs, and Notes is the right place for it, because raddle reads only the record. Keep it.

No further tool calls were made: my context already held everything this lens needed.
