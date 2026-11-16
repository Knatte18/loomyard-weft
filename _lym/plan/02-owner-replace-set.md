# Card 2 — owner-replace-set

**Edit:**
- `internal/boardengine#OwnerRequest`
- `internal/boardengine#SetOwners`
- `internal/boardengine#writeOwners`
- `internal/boardengine#TestSetOwners`
- `internal/boardengine#newOwnerFixture`
- `internal/boardengine/doc.go`
- `PATTERN.md`
- `pattern/PATTERN-owner-machine.md`

**Uses:**
- `internal/boardengine#ownerFixture.pushFromOther`
- `internal/boardengine#setOwnerOnOther`
- `internal/boardengine#OwnedElsewhere`

**Intent:** Give the owner write a replace set (D6), the field the rename sweep of `lym machine name` selects owners with.

`OwnerRequest` gains `Replace []string`: owner names to rewrite to `Machine`.
Its doc comment says the write itself selects, after its pull, every open beam project and the prime whose recorded owner is in the set, though that owner reads as another machine, and writes no other owner; `Takeover` stays unset for it.
In `writeOwners`, when `req.Replace` holds a non-empty name:
- the prime is written when `store.primeOwner` is in the set, appending `OwnerWritePrime` to `written` once even when `req.Prime` also asked for it;
- every task that is a beam project (`KindProject` with `Beam`), not done, whose recorded `OwnerMachine` is in the set gets `OwnerMachine = req.Machine` and joins `written` once, even when `req.Slugs` also names it;
- the match is on the recorded owner, never the effective one, so a member entry or a beam with no recorded owner is never selected;
- an empty string in the set matches nothing, so an unowned beam or a hub with no prime owner is never taken (acceptance: an empty `names` entry is no former name for the replace set).
A done entry keeps its owner, since the selection skips it.
The selection runs inside the write function, under the pull `PullThenCommitWritten` makes first, so an owner only origin held before the call is selected too.

In `SetOwners`, the commit message names the set when it is non-empty: `board: owner <machine> replacing <names joined by ", ">`, beside any slugs and prime the request names.
The post-push re-read already judges every written subject, so a replace-selected entry a competing origin write kept is reported in `Lost` with the owner that stands.
`SetOwners`'s doc comment states the replace-set case.

`TestSetOwners` (tier1) gains, as rows of its `outcomes` table where the shape fits:
- `ReplaceSetRewritesOpenBeamsAndPrimeUnderAFormerName`: prime owner `box`, `Replace: ["box"]`, machine `neve`; `boxed` and the prime read `neve` on origin, `beamer` (no recorded owner) stays empty, and a done beam project owned by `box` keeps `box`.
- `ReplaceSetLeavesOtherOwners`: `Replace: ["gone"]` writes nothing and leaves origin's head at the seed.
- `EmptyReplaceNameMatchesNothing`: `Replace: [""]` with prime owner `""` writes nothing.
And one subtest beside `CompetingWriteLosesToOrigins`: `ReplaceSetSelectsAnOwnerOnlyOriginHeld`, which pushes `beamer`'s owner `old` from the second clone through `pushFromOther` with `setOwnerOnOther`, then calls `SetOwners` with `Replace: ["old"]` and asserts `beamer` is recorded and reads `neve` on origin.
`newOwnerFixture` seeds one more entry for the done row: a done beam project `landed` owned by `box`, set on the store directly like `boxed`.
Existing rows name their slugs, so the added entry changes none of their assertions.

`internal/boardengine/doc.go`'s Owners section: "Without Takeover, SetOwners leaves an entry whose effective owner is another machine untouched" gains the replace-set exception, and the section states the selection rule above.

`pattern/PATTERN-owner-machine.md`'s bullet "It takes a project another machine owns only when `lym adopt` asks for it." becomes: it takes a project another machine owns only when `lym adopt` asks for it, or when the rename sweep of `lym machine name` names the project's recorded owner in its replace set of this machine's former names.
The background already says the name comes from the machine file before the hostname rule, so that sentence stays.
The declared change keeps the entry's overview line as it stands; the card leaves this line in `PATTERN.md` exactly as fenced:

```
- `PATTERN-owner-machine` — Starting, resuming, pushing or landing a run, or picking a pair's parent: only on the entry's owner machine, read through hubgeom's one `ParentOf`. — [background](pattern/PATTERN-owner-machine.md)
```

The field lets a write take owners that read as another machine's, bounded to recorded owners equal to a name in the set the caller passes, open beam projects and the prime only, and never a done entry or an empty owner.
Only `lym machine name` (card 4) sets it, with this machine's former names.

**ImpactSummary:** One new OwnerRequest field read only in writeOwners and the commit message; every existing SetOwners caller leaves it nil and keeps its behaviour.
