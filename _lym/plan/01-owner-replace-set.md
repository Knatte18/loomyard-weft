# Card 1 — owner-replace-set

**Edit:**
- `internal/boardengine#OwnerRequest`
- `internal/boardengine#SetOwners`
- `internal/boardengine#writeOwners`
- `internal/boardengine#newOwnerFixture`
- `internal/boardengine#TestSetOwners`
- `internal/boardengine/doc.go`
- `pattern/PATTERN-owner-machine.md`
- `PATTERN.md`

**Uses:**
- `internal/boardengine#ownerFixture.pushFromOther`
- `internal/boardengine#OwnedElsewhere`

**Intent:** Give the owner write a replace set, so `lym machine name`'s sweep (D6) can move this machine's owners from its former names to its new one.

`OwnerRequest` gains the field `ReplaceOwners []string`, documented as the owners the write replaces with `Machine`.
With it non-empty, `writeOwners` selects, on the store it loads after the write's own pull, every beam project that is not done and whose recorded `OwnerMachine` is in the set, and the prime when the recorded prime owner is in the set.
It records `Machine` on each selected entry even though that owner reads as another machine, and `Takeover` stays unset.
An empty string in the set matches nothing, so an empty `names` entry is never a former name.
A done entry, a task, a grouping project and an owner recorded under any name outside the set keep their owners.
The selection runs beside the existing `Slugs` and `Prime` handling, and an entry both name is written once.
Each selected slug, and `OwnerWritePrime` for the prime, joins `written`, so `SetOwners`' re-read after the push puts it in `Recorded` when the new owner stands, or in `Lost` with the standing owner when a competing origin write kept it.
The commit message is fixed before the pull, so a write with a replace set adds the phrase `replacing <set joined by ", ">` to its subject, after any named subjects: `board: owner neve for prime, replacing box`.
A write whose selection and named slugs write nothing commits nothing, as today.
`SetOwners`' and `OwnerRequest`'s doc comments state the replace set; `doc.go`'s Owners section states that `SetOwners` takes another machine's owner with `Takeover`, for `lym adopt`, or through the replace set, for the rename sweep.

`newOwnerFixture` seeds two more entries: `closed`, a done beam project owned by `box`, and `kept`, an open beam project owned by `osl`.
`TestSetOwners` (tier1) gains outcome rows:
- `ReplaceSetRewritesOwnersRecordedUnderAFormerName`: prime owner `box`, request `{Machine: "neve", ReplaceOwners: ["box", ""]}`;
  `Recorded` holds `boxed` and the prime, origin records `neve` on both, and `closed` stays `box`, `kept` stays `osl` and `beamer` records none.
- `ReplaceSetWithNoMatchCommitsNothing`: request `{Machine: "neve", ReplaceOwners: ["gone"]}` records nothing and leaves origin's head at the seed.
It gains one subtest, `ReplaceSetSweepsAnOwnerOnlyOriginHolds`: the second clone records `box` on `beamer` and pushes through `pushFromOther`, then the write with `ReplaceOwners: ["box"]` records `neve` on `beamer` at origin.

`pattern/PATTERN-owner-machine.md`'s bullet "It takes a project another machine owns only when `lym adopt` asks for it." becomes: it takes a project or the prime another machine owns only when `lym adopt` asks for it, or when `lym machine name`'s rename sweep names the recorded owner among this machine's former names.
The entry's overview line in `PATTERN.md` stays this line, the declared `changes` text:

```
- `PATTERN-owner-machine` — Starting, resuming, pushing or landing a run, or picking a pair's parent: only on the entry's owner machine, read through hubgeom's one `ParentOf`. — [background](pattern/PATTERN-owner-machine.md)
```

**ImpactSummary:** Adds one optional field to the owner write; every existing caller leaves it empty and keeps its behavior; the test fixture gains two entries no existing row asserts on.
