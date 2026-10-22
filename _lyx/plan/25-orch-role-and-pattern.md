# Card 25 — orch-role-and-pattern

**Edit:**
- `contracts/stencils/orch/orch-template-role.md`
- `PATTERN.md`

**Create:**
- `pattern/PATTERN-owner-machine.md`

**Intent:**
D9's orch reading and the declared PATTERN entry.
The orch role stencil's After landing section drops "or whose pair is gone" from the wake rule, so a missing pair alone never reads as landed, and gains a rule: a run whose board entry or `lyx tender status` names another machine as its owner is that machine's run, neither stalled nor landed, and the orch never resumes or cleans it up here; `lyx adopt <slug>` from this prime is the takeover, and the operator decides it.
The Investigating a stop section names a Finalize halt that opens `owner check:` as an owner halt whose message names the adopt forms, which the orch relays to the operator.
`pattern/PATTERN-owner-machine.md` holds the entry's full declared text, verbatim from the overview's `## PATTERN changes` section, then its clauses as bullets and an Enforcement section naming `internal/machinename`'s sole-source scan, the reconcile, batten, loom, InnerRun and Finalize tests that reach each check, and review for the rest.
`PATTERN.md` gains, in its Fabric and git section after `PATTERN-origin-reconcile`, this entry line, under `pattern.MaxEntryLineChars`:

```markdown
- `PATTERN-owner-machine` — Starting, resuming or landing a run, bar a Finalize past its parent-side merge, or acting after a wake: only where its entry reads owned by no other machine; owner set only by `boardengine`'s setter; origin verdicts in `ReconcileOrigin`; name only from `internal/machinename`. — [background](pattern/PATTERN-owner-machine.md)
```

`PATTERN.md` already stands above its headroom line, so, as the decision record's "Added after Discussion: PATTERN.md must not grow for PATTERN-owner-machine" entry decides, the branch cuts as many bytes from the wording of other entry lines as the new line adds, and the overview ends no larger than at the merge base; `lyx loom lint-pattern` passes.
Each cut only drops or shortens words of a line's wording: it removes no clause, named package, symbol, operand, exception or `(test)` marker, and moves no detail out of the overview, so every rule the line states stands; the commit body lists each cut line before and after, so review checks the rules line by line.
The commit carries a `Pattern-Departure:` trailer citing `PATTERN-owner-machine`'s declared text and naming `pattern.MaxEntryLineChars` and the headroom line as the reasons the overview line is a short form whose full text lives in the background file.

**ImpactSummary:** the orch role stencil and the PATTERN overview with one new background file; no code changes.

**Commit:** 25: orch and PATTERN: a run owned elsewhere is that machine's
