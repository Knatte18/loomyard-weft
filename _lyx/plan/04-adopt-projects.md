# Card 4 — adopt-projects

**Edit:**
- `internal/adoptcli#request`
- `internal/adoptcli#adopt`
- `internal/adoptcli#namedEntries`
- `internal/adoptcli#takeOverSlug` -> `func takeOverSlug(deps adoptDeps, req request, rec *fabricengine.Mutations, report fabricengine.OriginReport, owners boardengine.Owners, project string, out *slugOutcome) bool`
- `internal/adoptcli#resumeSlug`
- `internal/adoptcli#parentWarning`
- `internal/adoptcli#liftOwnerPauses`
- `internal/adoptcli#resumeHaltedChildren`
- `internal/adoptcli#adoptedUnder`
- `internal/adoptcli#refuseStandingMove`
- `internal/adoptcli#wouldStop`
- `internal/adoptcli#slugOutcome`
- `internal/adoptcli#Command`
- `internal/adoptcli#AdoptAs` -> `func AdoptAs(prime *lyxcwd.Location, machine string, slugs []string, keepLocal, dropLocal bool) (Outcome, error)`
- `internal/adoptcli#world.deps`
- `internal/adoptcli#TestAdopt`
- `internal/adoptcli_test#TestAdoptIntegration_SecondCloneTakesOver`
- `internal/adoptcli/doc.go`
- `docs/overview.md`

**Create:**
- `plan:internal/adoptcli#movedRuns` -> `func movedRuns(deps adoptDeps, owners boardengine.Owners, adopted []string, rootAdopted bool) ([]string, error)`

**Delete:**
- `internal/adoptcli#primeSubject`
- `internal/adoptcli#adoptNeeds`
- `internal/adoptcli#needs`
- `internal/adoptcli#needs.subjects`
- `internal/adoptcli#adoptNeeds.parentOf`
- `internal/adoptcli#adoptNeeds.of`
- `internal/adoptcli#adoptNeeds.closure`
- `internal/adoptcli#adoptRefusal`
- `internal/adoptcli#needs.describe`
- `internal/adoptcli#subjectLabel`
- `internal/adoptcli#adoptCall`

**Intent:**
D6: `lyx adopt <project>...` and `lyx adopt root` replace `lyx adopt <task>` and `lyx adopt --prime`.

- The arguments are beam project slugs and `boardengine.RootSubject` (`root`), which names the prime; `--all` names root and every open beam project; `--keep-local` and `--drop-local` stay; `--prime` is removed from `Command` and `request.Prime` becomes `request.Root`, set by the `root` word.
  An entry whose slug is `root` is shadowed by the word and cannot be named, which `doc.go` states.
- `namedEntries` refuses before anything runs: a task, naming `lyx adopt <AdoptSubject>` (its beam or `root`); a grouping project, naming `lyx adopt <AdoptSubject>` to move its tasks and `lyx beam create <slug>` to make it a beam; a note and an unknown slug as today; a done entry as today.
  The test that reaches each is a row of `TestAdopt` with that entry named.
- An adopt moves the named projects alone: each one's owner (`prime_owner` for root), its pair, and the runs whose parent is its worktree.
  `movedRuns` lists, sorted, every task and beam project that is not done, holds a batten run record that is not done (`deps.RunRecord`), and whose `deps.ParentOf` answer is an adopted beam, or `""` when root is adopted, plus each adopted beam's own pawl run.
  A nested beam keeps its own owner: `lyx adopt root` moves root's tasks and the tasks of grouping projects below root, never a beam's members, since their `ParentOf` answer is that beam.
- Step 1 and step 2 run over the adopted beams and the moved runs: a moved slug the report reads owned_elsewhere with local commits or a live pair stops as a named slug stops today, and the pair of each, absent here, is added when the entry holds a run status or is an owned beam.
  A stop's way forward names the adopt that moved the slug, never the slug itself, which `namedEntries` refuses for a task: `takeOverSlug` takes the adopted project, the beam or `root`, and its text reads `lyx adopt <project> --keep-local` and `lyx adopt <project> --drop-local`.
- The refusals that a parent or a member reads owned by another machine go (`adoptRefusal` and its graph helpers), since no task has an owner and a beam may be owned apart from its parent; `refuseStandingMove` and `wouldStop` keep their checks over the new slug sets.
- One owner write records this machine for every adopted beam not stopped and, with `Root` set or when the board records no prime owner, for the prime.
- After the owner write, each moved run is resumed as a detached first pass (`deps.ResumeRun`), after its owner pause is lifted, whenever its entry's effective owner, read from the re-read board, is this machine; a beam's pawl run is resumed only when this machine also owns the beam's parent, whichever of the two the adopt named, so adopting root then G and G then root both leave G's pawl run resumed here.
  A run whose entry reads owned by another machine, such as a nested beam's pawl run under an adopted root, is not resumed and is reported on its own outcome as `owned elsewhere` with its owner, never as an error.
  `parentWarning` names `lyx adopt <AdoptSubject of the parent>`, `lyx adopt root` for the prime.
- `resumeHaltedChildren` and `adoptedUnder` read root's adoption from `Root`; the dobby start and the finalize suite stay as they are (cards 7 and 24 narrow and extend them).

Bound: until the same machine owns a beam and its parent, the beam's pawl run gets no batten pass here, reported owned elsewhere; `lyx adopt <beam>` on the parent's owner, or `lyx adopt <parent>` on the beam's owner, resumes it.

`TestAdopt` loses its parent- and member-refusal rows and gains rows reaching: a task refused naming its beam, a root task refused naming `root`, a grouping project refused naming its beam and `lyx beam create`; `root` moving a root task's run and leaving a nested beam's member alone; a beam moving its member's run; a beam's pawl run resumed when root is adopted with the beam already owned here, and when the beam is adopted with root owned here; a nested beam's pawl run under an adopted root reported owned elsewhere with no error; a moved task whose pair holds local commits stopped naming `lyx adopt <its beam> --keep-local` and `--drop-local`, never the task; `--all` naming root and every open beam project and no task or grouping project.
`world.deps` serves `ParentOf` and `RunRecord` for those fixtures.
`TestAdoptIntegration_SecondCloneTakesOver` adopts `root` instead of its task and still asserts the task's pair added and its pause lifted, now with the prime's owner recorded and no task owner.
`adoptcli/doc.go`, `Command`'s help and `docs/overview.md`'s adopt line describe the verb above.

**ImpactSummary:** adopt's argument contract and run selection change inside the package; `AdoptAs` is test-only and its one caller is on this card; no other package calls adopt.

**Commit:** 4: `lyx adopt` takes beam projects and `root`, and moves the runs under them
