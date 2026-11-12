# Card 1 — owner-replace-set

**Edit:**
- `internal/boardengine#OwnerRequest`
- `internal/boardengine#SetOwners`
- `internal/boardengine#writeOwners`
- `internal/boardengine#TestSetOwners`
- `internal/boardengine/doc.go`
- `//pattern/PATTERN-owner-machine.md`
- `//PATTERN.md`

**Uses:**
- `internal/boardengine#newOwnerFixture`
- `internal/boardengine#ownerFixture.originOwners`
- `internal/boardengine#ownerFixture.pushFromOther`
- `internal/boardengine#setOwnerOnOther`
- `internal/boardengine#failOnSleep`

**Intent:** Give the owner write a replace set, so the rename sweep (D6) rewrites every owner recorded under this machine's former names, selected after the write's own pull.

`OwnerRequest` gains the field `Replace []string`, documented as the replace set: the former names whose owners the write rewrites to `Machine`.
With it set, `writeOwners` selects, on the store it loads after the pull, every open beam project whose recorded `OwnerMachine` is in the set, and the prime when `primeOwner` is in the set.
It rewrites each to `req.Machine` although that owner reads as another machine, so the `OwnedElsewhere` check never skips a selected entry, and `Takeover` stays unset.
An empty entry of the set matches nothing, so an unrecorded owner is never selected (acceptance: an empty `names` entry is no former name for the replace set).
A done entry is never selected, and an owner recorded under a name outside the set is left alone.
The slugs it selects join `written` beside those `Slugs` and `Prime` name, each once, so the existing re-read after the push decides `Recorded` and `Lost` for them as for any other written entry.
With `Replace` set the commit subject reads `board: owner <machine> replacing <names>`, the non-empty names joined with `, `, since the selected slugs are known only after the pull.

`TestSetOwners` (tier1) gains, in its `outcomes` table, the row `EmptyReplaceEntryMatchesNoUnrecordedOwner`:
a fixture with no prime owner and `Replace: []string{""}` records nothing, leaves `beamer` and the prime unowned and leaves origin's head at the seed.
It gains a subtest `ReplaceSetRewritesFormerNameOwnersAfterThePull`:
the second clone, through `pushFromOther`, records `old` as the prime owner and as `beamer`'s owner and marks a beam project recorded under `old` done, at origin only;
`SetOwners` with `Machine: "neve"`, `Replace: []string{"old"}` and no `Takeover` then records `beamer` and `OwnerWritePrime`.
Origin afterwards reads `neve` for `beamer` and the prime, `box` for `boxed`, and `old` on the done entry, and its head subject is `board: owner neve replacing old`.

`internal/boardengine/doc.go`'s `# Owners` passage says the owner setter writes the owners the replace set selects after its pull, beside the named slugs, and that only `lym machine name` passes a replace set.

`pattern/PATTERN-owner-machine.md`'s bullet "It takes a project another machine owns only when `lym adopt` asks for it." gains the second case: or when its owner is in the replace set of `lym machine name`'s rename sweep.
The entry's overview line in `PATTERN.md` stays as fenced below, since the change is the background's alone:

```markdown
- `PATTERN-owner-machine` — Starting, resuming, pushing or landing a run, or picking a pair's parent: only on the entry's owner machine, read through hubgeom's one `ParentOf`. — [background](pattern/PATTERN-owner-machine.md)
```

**ImpactSummary:** Adds one opt-in field to the owner write; every existing caller leaves it unset and keeps today's behavior.
