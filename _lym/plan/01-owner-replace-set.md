# Card 1 — Owner write replace set

**Edit:**
- `internal/boardengine#OwnerRequest`
- `internal/boardengine#SetOwners`
- `internal/boardengine#writeOwners`
- `internal/boardengine#TestSetOwners`
- `internal/boardengine#newOwnerFixture`
- `internal/boardengine#ownerFixture`
- `pattern/PATTERN-owner-machine.md`
- `PATTERN.md`

**Extends:**
- `internal/boardengine/doc.go`

**Uses:**
- `internal/boardengine#ownerFixture.pushFromOther`
- `internal/boardengine#setOwnerOnOther`
- `internal/boardengine#isDone`
- `internal/boardengine#OwnedElsewhere`

**Intent:** Give the owner write the replace set of D6, so a rename sweep keeps owners without `Takeover`.

`OwnerRequest` gains the field `Replace []string`, the owner names the write replaces with `Machine`.
Its doc comment says the write selects the entries itself, after its pull, and that only the rename sweep of `lym machine name` passes it.

`writeOwners` runs the selection after `store.Load`, which runs after the pull, so an owner only origin held is selected.
With a non-empty `Replace` it rewrites to `req.Machine`:

- the prime owner, when `store.primeOwner` is in the set;
- every beam project that is not done (`isDone`) and whose recorded `OwnerMachine` is in the set.

It reads the recorded owner, never the effective one, so a member of a beam and an unowned beam are never selected.
An empty string in `Replace` selects nothing, so an entry with no recorded owner is never taken;
an entry of `Replace` equal to `req.Machine` selects nothing either.
The selection ignores `Takeover`, which stays unset in the sweep's request:
the selected owner reads as another machine, and the write goes through because the set names it.
Every selected name joins `written`, the prime as `OwnerWritePrime`, so `SetOwners` judges it after the push like any other:
one whose owner reads `Machine` goes to `Recorded`, and one a competing origin write kept goes to `Lost` with the standing owner.
The selection runs beside `Slugs`, `Prime` and `Claim`, which keep their rules; a slug both named and selected is written once.
No owner outside the set is written by the selection, and a done entry keeps its recorded owner.

`SetOwners` builds its commit message before the write runs, so a request with a replace set names the set:
`board: owner <machine> for <slugs and prime>, owners recorded as <names>`, the names comma-joined in the request's order,
and `board: owner <machine> for owners recorded as <names>` when the request names no slug and no prime.
A request with an empty `Replace` keeps today's message.
`SetOwners`'s doc comment gains the replace set.

What this lets through: an owner write over another machine's name with no `Takeover`.
Bound: only for an owner recorded under a name the caller hands in, only on the prime and open beam projects, and never an empty owner.
The caller, `lym machine name`, hands in this machine's former names alone.

Tests, all rows or subtests of `TestSetOwners`, tier1 like the rest of it:

- `newOwnerFixture` seeds two more beam projects, and `ownerFixture`'s doc comment lists them:
  `shipped`, done and owned by `box`, and `theirs`, open and owned by `far`.
- Outcome row, prime owner `box`, request `{Machine: "neve", Replace: []string{"box"}}`:
  `Recorded` holds the prime and `boxed`; origin reads `neve` for both, `box` for `shipped`, `far` for `theirs` and no owner for `beamer`;
  the origin head's subject is `board: owner neve for owners recorded as box`.
- Outcome row, prime owner `neve`, request `{Machine: "neve", Replace: []string{"", "neve"}}`: nothing is recorded and origin's head stays at the seed, so the unowned `beamer` is not taken.
- Subtest: `pushFromOther(setOwnerOnOther("beamer", "box"))` runs before the call, so only origin holds that owner;
  the request `{Machine: "neve", Replace: []string{"box"}}` records `beamer` with `boxed`.
- Subtest: `BeforePush` pushes `setOwnerOnOther("boxed", "far")` from the other clone;
  the same request reports `boxed` in `Lost` with `far`, and origin keeps `far`.

`internal/boardengine/doc.go`'s Owners section gains the replace set after the `Takeover` sentences:
what it selects, that it needs no `Takeover`, and that only `lym machine name` passes it.
The sentence "Only `lym adopt` passes Takeover." stays.

`pattern/PATTERN-owner-machine.md`: the bullet clause "It takes a project another machine owns only when `lym adopt` asks for it." becomes two cases:
when `lym adopt` asks for it, and when the rename sweep of `lym machine name` hands in a replace set naming its recorded owner, this machine's former names.
The entry's overview line in `PATTERN.md` and the background's opening line keep the text below; this card confirms both read exactly so and changes neither when they do.

```
- `PATTERN-owner-machine` — Starting, resuming, pushing or landing a run, or picking a pair's parent: only on the entry's owner machine, read through hubgeom's one `ParentOf`. — [background](pattern/PATTERN-owner-machine.md)
```

**ImpactSummary:** Adds one optional field to the owner write; every existing caller leaves it empty and behaves as before, and the fixture's two new seed entries are named by no existing row.

**Commit:** 1: boardengine: the owner write takes a replace set for the rename sweep
