# Card 1 — owner-replace-set

**Edit:**
- `internal/boardengine#OwnerRequest`
- `internal/boardengine#SetOwners`
- `internal/boardengine#writeOwners`
- `internal/boardengine#TestSetOwners`
- `PATTERN.md`
- `pattern/PATTERN-owner-machine.md`

**Extends:**
- `internal/boardengine/doc.go`

**Uses:**
- `internal/boardengine#newOwnerFixture`
- `internal/boardengine#ownerFixture.pushFromOther`
- `internal/boardengine#failOnSleep`

**Intent:**
Give the owner write the replace set D6 names, so a rename sweep rewrites every owner recorded under this machine's former names, selected after the write's own pull.

`OwnerRequest` gains a field `Replace []string`, the replace set: owner names the write rewrites to `Machine`.
With `Replace` set, `writeOwners`, which runs after the pull under the board write lock, selects every open beam project whose recorded `owner_machine` is in the set, and the prime when the recorded `prime_owner` is in the set.
It rewrites each to `req.Machine` although that owner reads as another machine, and appends each to `written`, so the re-read after the push files it under `Recorded` or `Lost` as for any other written entry.
The selection compares the recorded owner, not the effective one, so a member reading its beam's owner is never written.
An empty string in the set selects nothing, so an empty `names` entry never reads as a former name.
A done entry keeps its recorded owner, as the owner write never writes one.
`Takeover` stays unread by the selection, and an owner under any name outside the set is left as it stands.
`Replace` combines with `Slugs` and `Prime` as an added selection; the sweep sends it with neither.
Bound: the field lets the write overwrite an owner that reads as another machine without `Takeover`, but only an owner whose recorded name the caller puts in the set; `lym machine name` puts only this machine's former names there.
The commit message for a write with `Replace` set reads `board: owner <machine> replacing <names>`, the set's non-empty names joined by `, `, since the selected subjects are known only after the pull.

`TestSetOwners` (tier1, its file's existing tier) gains a subtest for the replace set over `newOwnerFixture` with prime owner `box`.
Before the write, `pushFromOther` adds owners origin alone holds: `beamer` recorded under `box`, a done beam project recorded under `box`, and a beam project recorded under another name.
The request is `OwnerRequest{Machine: "neve", Replace: []string{"box", ""}}` with `Sleep` from `failOnSleep`.
It asserts `Recorded` holds `boxed`, `beamer` and `OwnerWritePrime`, `Lost` is nil, origin records `neve` for `boxed`, `beamer` and the prime, and the done beam and the other name's beam keep their owners.
It asserts the origin head subject is `board: owner neve replacing box`.

`internal/boardengine/doc.go`'s `# Owners` section gains the replace set: what it selects, after the pull, and that it ignores an empty name and a done entry.

`pattern/PATTERN-owner-machine.md`'s Owner machine section changes the line saying the owner setter takes a project another machine owns only when `lym adopt` asks.
It names the rename sweep's replace set as the second case, writing only owners recorded under this machine's former names.
The section's `Rule:` line stays as it reads.
The overview line in `PATTERN.md` keeps its text, the entry's declared `changes` text:

```markdown
- `PATTERN-owner-machine` — Starting, resuming, pushing or landing a run, or picking a pair's parent: only on the entry's owner machine, read through hubgeom's one `ParentOf`. — [background](pattern/PATTERN-owner-machine.md)
```

**ImpactSummary:** boardengine's owner write gains an opt-in selection; every existing caller leaves `Replace` nil and writes as before.

**Commit:** 1: Owner write gains a replace set that sweeps former owner names after its pull
