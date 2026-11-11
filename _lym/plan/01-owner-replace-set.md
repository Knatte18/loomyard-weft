# Card 1 — owner-replace-set

**Edit:**
- `internal/boardengine#OwnerRequest`
- `internal/boardengine#OwnerWrite`
- `internal/boardengine#SetOwners`
- `internal/boardengine#writeOwners`
- `internal/boardengine#TestSetOwners`
- `internal/boardengine/doc.go`
- `pattern/PATTERN-owner-machine.md`
- `PATTERN.md`

**Uses:**
- `internal/boardengine#newOwnerFixture`
- `internal/boardengine#ownerFixture.pushFromOther`
- `internal/boardengine#ownerFixture.originOwners`
- `internal/boardengine#failOnSleep`
- `internal/boardengine#setOwnerOnOther`
- `internal/boardengine#isDone`

**Intent:**
`boardengine.OwnerRequest` gains the replace set (D6), a struct field `Replace []string`.
Its doc says: the names whose recorded owners the write takes over, this machine's former names, which only `lym machine name`'s sweep passes.

`SetOwners` refuses, beside its existing claim check and before any pull, a request that sets `Replace` together with `Slugs`, `Prime`, `Claim` or `Takeover`.
This is a fail-loud wiring guard: the verb's sweep, the only caller of `Replace`, sets none of them, so it takes no reaching test.

`writeOwners` applies `Replace` after its load, which runs inside `SetOwners`' pull, so an owner only origin held before the pull is selected too.
With `Replace` holding at least one non-empty name:
- the prime is written to `req.Machine` when the recorded prime owner (`store.primeOwner`) is a non-empty member of `Replace`, and `OwnerWritePrime` joins `written`;
- every open beam project (kind project, beam flag set, not `isDone`) whose recorded `OwnerMachine` is a non-empty member of `Replace` gets `OwnerMachine = req.Machine` and joins `written`.
The selection compares the recorded owner, never the effective one, and skips the `OwnedElsewhere` check, so it writes over an owner that reads as another machine with `Takeover` unset.
It writes no other owner: a done entry, an unowned beam, an owner recorded under any name outside the set, a task and a grouping project keep what they hold.
An empty entry in `Replace` selects nothing, so an unowned beam (`OwnerMachine` "") and an unrecorded prime are never taken (acceptance criterion on empty entries).
The re-read after the push judges each selected slug as it judges a named one, so a competing origin write that kept an entry reports it in `OwnerWrite.Lost`.
The `writeOwners` doc comment names the replace selection beside "the owners req asks for".
`OwnerWrite.Recorded`'s field doc names the slugs and `OwnerWritePrime` the replace set selected, beside those the request named.

The commit message: with `Replace` holding a non-empty name, the subject is `board: owner <machine> replacing <the non-empty names joined by ", ">`, for example `board: owner neve replacing box, old`; without it the subject stays `board: owner <machine> for <subjects>`.

Bound: the replace set lets the write through an owner that reads as another machine only where the recorded owner string is one of the names the caller passes, on open beam projects and the prime; `Takeover` stays the only way to take any other owner, and `lym adopt` stays its only caller.

Tests, tier 1, in `TestSetOwners`:
- an outcome row `ReplaceSetWithNoMatchingOwnerWritesNothing`: prime owner `neve`, `OwnerRequest{Machine: "neve", Replace: []string{"old"}}`, nothing recorded, no lost entry, origin head stays at the seed;
- a subtest `ReplaceSetSelectsAfterThePull`: fixture with prime owner `box`;
  `pushFromOther` appends, at origin only, a beam project `oldbeam` owned by `box`, a beam project `farbeam` owned by `far` and a done beam project `shipped` owned by `box` (status `StatusDone`), and leaves `beamer` an open beam with no owner;
  `SetOwners(Machine "neve", Replace {"box", "", "old"}, Sleep failOnSleep)` records `prime`, `boxed` and `oldbeam` (compared sorted), loses none;
  origin then holds `neve` for `boxed`, `oldbeam` and the prime, `far` for `farbeam`, `box` for `shipped`, `""` for `beamer`, and the head subject `board: owner neve replacing box, old`;
- a subtest `ReplaceSetLosesToACompetingOriginWrite`: fixture with prime owner `neve`; `SetOwners(Machine "neve", Replace {"box"})` with a `BeforePush` that `pushFromOther`s `setOwnerOnOther("boxed", "far")`; the result records nothing and loses `boxed` to `far`, and origin holds `far` for `boxed`.

`internal/boardengine/doc.go`'s owner-write paragraph changes its commit line to name both subjects, `"board: owner <machine> for <slugs>"` and, with a replace set, `"board: owner <machine> replacing <names>"`.
It gains the replace set: what it selects, after the pull, over which owners, that it goes with no slug, prime, claim or takeover, and that only `lym machine name` passes it, beside "Only `lym adopt` passes Takeover".

`pattern/PATTERN-owner-machine.md`'s clause "It takes a project another machine owns only when `lym adopt` asks for it." becomes:
"It takes a project or the prime another machine owns only when `lym adopt` asks for it, or when `lym machine name`'s sweep names that recorded owner among this machine's former names."
The declared `changes` entry keeps its overview line in `PATTERN.md` as:

```
- `PATTERN-owner-machine` — Starting, resuming, pushing or landing a run, or picking a pair's parent: only on the entry's owner machine, read through hubgeom's one `ParentOf`. — [background](pattern/PATTERN-owner-machine.md)
```

**ImpactSummary:** a new optional field on `OwnerRequest` and a doc line on `OwnerWrite`; every existing caller (adopt, beam, batten, orch, dobby geometry) leaves `Replace` empty and keeps today's behaviour and commit subject, so only boardengine's own code, test and doc change.

**Commit:** 1: boardengine: let an owner write take over this machine's former names
