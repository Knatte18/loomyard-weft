# Card 25 — pattern-owner-machine

**Edit:**
- `PATTERN.md`
- `pattern/PATTERN-owner-machine.md`

**Intent:** Record the declared change to `PATTERN-owner-machine`: owners read through the machine registry, a Finalize past its merge finishes on the owner, and a catch-up from origin comes first after an absence.

The entry's overview line in `PATTERN.md` becomes:

```
- `PATTERN-owner-machine` — Starting, resuming, pushing or landing a run, a Finalize past its merge included, or picking a pair's parent: only on the entry's owner machine, read through hubgeom's one `ParentOf` and Bolt's machine registry; after an absence, a catch-up from origin first. — [background](pattern/PATTERN-owner-machine.md)
```

The background file's rule text matches, and its sections change with the code:
"Owner machine" says every comparison of a recorded name with this machine goes through `boardengine.OwnedElsewhere` over the registry, that a former name of a registered machine reads as that machine, and that the rest of a Finalize past its parent-side merge finishes on the run's owner;
"Origin reconcile first" becomes the catch-up after an absence (a sleep the dobby sees, a dobby start, a hand pass) and drops the origin-owed marker;
"Passes" drops the marker from the continue-pass refusal and the full-pass rule;
"Enforcement" names `TestReadsAsHere`, `TestRegistryCheck`, `TestCatchUp` and `TestRun_CatchUp` beside the tests it names.
`machinename` stays this machine's only name source, with the id beside the name.

**ImpactSummary:** Two documentation files; no code or test changes.
