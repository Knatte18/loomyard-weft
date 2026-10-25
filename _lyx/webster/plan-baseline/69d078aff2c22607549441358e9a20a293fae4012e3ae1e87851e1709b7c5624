# Card 39 — adopt-refusals

**Edit:**
- `internal/adoptcli#adopt`
- `internal/adoptcli#adoptDeps`
- `internal/adoptcli#outcome`
- `internal/adoptcli#hubDepsAs`
- `internal/adoptcli#newWorld`
- `internal/adoptcli#TestAdopt`
- `internal/adoptcli/doc.go`

**Create:**
- `internal/adoptcli#wouldStop`
- `internal/adoptcli#adoptRefusal`

**Intent:**
Refuse an adopt that would leave no dobby here to drive what it takes over, and end every adopt by starting the dobbies (D2, and the Added-after-Discussion entry deciding the refusals before any mutation).
`wouldStop` answers, from the read-only part of step 1 alone (`ownedElsewhere`, `stoppedBranches` and `LivePair`), whether `takeOverSlug` would stop the entry; `adopt` computes the entries step 1 does not stop and calls `adoptRefusal` ahead of the `takeOverSlug` loop, so a refused call drops no local commits, adds no pair and writes no owner.
`adoptRefusal` refuses an entry it records an owner for, a task or a beam, whose parent (through `deps.ParentOf`, the prime for none) reads owned by another machine, unless the same call adopts the parent: the parent is among the entries going on, or the prime is named for the prime; and it refuses an adopted beam, the prime included, while a member that is not done records another machine as its owner, naming those members, unless the call adopts them too.
A parent or member the same call stops at step 1, as a diverged or live pair, does not count as adopted.
Each refusal names one call that meets both rules: `lyx adopt` with every slug the two rules reach from the named entries, followed to a fixed point, and `--prime` when the prime is among them, so following it meets no second refusal; when an entry the two rules reach would itself stop at step 1 (local commits origin lacks, or a live pair), the named call also carries that stop's own answer, `--keep-local`, for local commits and for a live pair alike, since `takeOverSlug`'s step-1 arm passes both under `--keep-local` and its own way forward offers that flag for "a live driver", while a slug pause never makes adopt's `LivePair` read not live (`hubOnlyPaused` counts only a hub entry), so the named call passes step 1 for it; the text is `lyx adopt: <entry> needs <parent or members> adopted with it, since this machine's dobby drives only runs whose parent is its own worktree; way forward: run \`<call>\``.
Accepted bound: a parent the batched owner write loses to a race while it records the child is not seen by the refusal, and the child keeps `parentWarning`'s warning naming `lyx adopt <parent>`.
`adoptDeps` gains `StartDobbies`, filled by `hubDepsAs` with `dobbyengine.EnsureLive(hubgeom.DobbyStart(prime))`, which `adopt` calls last, after the resumes, so a dobby runs wherever the adopted runs' parent now is; a failure is the outcome's `dobby_warning` naming `lyx beam start`, and `outcome` gains that field.
The package doc lists the two refusals and the last step.
Tests: `TestAdopt` (the fake world `newWorld`) gains the refusal rows: a task whose beam another machine owns is refused naming `lyx adopt <beam> <task>`; a task of the prime another machine owns names `lyx adopt --prime <task>`; the same call naming the parent passes; naming the parent while step 1 stops the parent on local commits is refused naming a call carrying `--keep-local`, and that call passes; naming the parent while step 1 stops the parent as a live pair (the fake world's `LivePair` reading it live) is refused naming a call carrying `--keep-local`, and that call passes with the pair still live; a beam with a not-done member owned elsewhere names the member; `--prime` with a not-done prime member owned elsewhere names that member; the named call itself passes; a refused call records no owner, adds no pair and drops nothing; and a row asserting `StartDobbies` runs last and a failure becomes the warning.

**ImpactSummary:** Two refusals move ahead of adopt's first mutation and one last step starts the dobbies; parentWarning stays for the race bound.
