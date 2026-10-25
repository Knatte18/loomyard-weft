```yaml
lens: leanness
outcome: reported
```

Leanness review of decision-record.md and support-log.md: what to move or cut, plus anything else noticed.

**Findings**

1. MEDIUM — Constraints, first bullet (the PATTERN checklist). It restates each PATTERN entry next to the decision that already carries it, so the same rule appears twice in the record.
   - The `entry_gate` and ungated `Drift-Await` points repeat D2.
   - The capture role and logging repeat D4.
   - The scope-record-through-a-verb point repeats D3.
   - The `drift-verify` gate repeats D6.

   Apply the completeness-before-leanness test first. Keep only the requirements that no decision states: the `_lyx/drift/` versus `.lyx/drift/` split and that only `internal/lyxdirs` names those paths; that `lyx drift` verbs stay allowed to agents; and that the stencils render the parent and edit directives. Cut the rest down to a line saying the design departs from no PATTERN entry. The entry-by-entry check is deliberation and fits the support log.

2. MEDIUM — D4 Rationale, and the "rather than Discussion-Write" sentence in D2. Both argue against an option that was not chosen: "issue #609 rules out a capture in a separate checkout, and the board entry rejects one". The compaction rule keeps rejected alternatives out of `## Decisions`. Keep the requirement itself, "the capture runs in the task worktree, never a separate checkout", as a constraint. Move the rejection narrative to the support log's Rejected alternatives.

3. MEDIUM — Constraints, the Models bullet's VSTest exit-code claim. "VSTest's `dotnet test` exits 0 when no test failed" is a fact the capture's pass rule (D4) rests on, and the record names no source or version for it. The issue #609 facts beside it do have a source. Either cite the source and version (the dotnet test docs, the NUnit3TestAdapter's mapping of Inconclusive) or mark it a Models-side check under Open risks. As written it is an unsupported premise.

4. LOW — Auto-mode assumptions. Most bullets repeat values D1, D4 and D6 already state: the default models, the budget of 8, the skip file and the logs location. Say-it-once says reference the decisions ("the defaults in D1, the skip interface in D1 and the logs location in D4 are auto-picks") instead of restating the values. Otherwise a later change to a default has to be made in two places.

5. LOW — Notes for the plan writer. Its first two bullets point at files `relevant.md` already lists with the same reason: `entries_command.go`, `entrygate.go` and darn's `Describe` row. The spec puts the code the writer read in `relevant.md`, never in the record. Cut those two bullets and keep the `Name*` constants hint.

6. LOW — Scope, In list, "Tests with fake capture and compare commands; nothing here needs Models". The test approach belongs in `## Acceptance criteria`, since the record has no testing section. "Nothing here needs Models" already appears under Scope, Out. Move or cut it.

7. LOW — Constraints, the `pairteardown` bullet. Half of it repeats D5's dead-process Stuck. Keep the fact about the existing code ("pair teardown ends the capture through `runproc.EndSparing`") and drop the restated halt.

8. LOW, emphasis — the support log's Interview. It is two turns, and the second only points at the ledger, which is fine for an autonomous run. The lens-review line still has to land at its last turn.

**Tool calls**

None; the reviewed files, the rubric and the spec were all in context.
