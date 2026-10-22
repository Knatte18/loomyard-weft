# Owner machines and `lyx adopt`: one machine takes over from another

Two machines of one operator can now work on a hub, never at once, and one takes over from the other with an explicit verb.
Before, a second machine resumed a run another machine had started with no check, read a missing pair as a landed run, and a machine waking from a suspend drove its old runs on beside the adopter.

Closes #741
Closes #742

## What changes

- **Owner records.**
  Every task and the prime records an owner machine on Bolt, named by the new `internal/machinename` leaf (the hostname; an empty one is an error).
  `lyx board`, `lyx batten status` and `lyx tender status` show it.
  `lyx board upsert` refuses a changed `owner_machine`; a run-held entry's owner changes only through adopt.
  A board replay of two owner writes keeps origin's value and reports the local one lost.
- **Owner check on start and resume.**
  `lyx batten run` and `lyx loom start` on a machine that does not own the entry refuse before creating or resuming anything, naming the owner and `lyx adopt <slug>`.
  A start records this machine as owner; a batten resume of an unowned run-held task records it first.
  With origin unreachable, a start or an owner-writing resume is refused, a task the local Bolt shows as owned here resumes with a warning, and one it shows as owned elsewhere is refused.
- **`lyx adopt`.**
  `lyx adopt <slug>...`, `--prime` and `--all`, with `--keep-local` and `--drop-local` to settle a slug's local commits.
  It brings the repos the entries need up to date from origin, records this machine as owner, pushes, and resumes the adopted runs here (`battencli.ResumeInWindow`).
  `hubforge.CloneSecond` and `fabricengine.DropLocal` support it.
- **Origin reconcile.**
  `fabricengine.ReconcileOrigin` reports `owned_elsewhere` with the owner name, in hub and run scope, and replays no records branch for such an entry.
  The hibernate wake and `lyx orch start` pause a run owned elsewhere; InnerRun re-checks the owner after a sleep or a pause hold ends, releases the run's parent lock, and pauses the pair through a new tender pause seam labelled with the owner.
  The orch stencil reads a run owned elsewhere as that machine's.
- **Landing.**
  `internal/landingshed` Finalize lands a run only on the machine that owns the run and its parent, checked before the parent-side merge.
  A failed check halts with a note naming the owner and the ways forward.
  A Finalize already past its parent-side merge (`fabricengine.ParentHoldsTaskWork`) skips the checks.
- **Open-Window per machine (#742).**
  A resume reopens the window when no client is attached (`reedengine.ClientAttached`).
- **Docs.**
  New `pattern/PATTERN-owner-machine.md`, with edits to the CLI and leaf-package patterns, the module `doc.go` files and the orch and board skills.

## Design decisions

- The owner is the hostname.
  Two machines with the same hostname read as one owner, and a renamed host refuses its own runs until it adopts them.
- Adopt takes the entries it names and nothing else; adopting the prime does not move its children.
- A run that must pause because another machine owns it is paused, not halted, so the pause holds until adoption back.
- Beams and a parent owner landing another machine's run from origin are out of scope (note `orch-any-worktree`).
  Ending a run's processes on the machine it left is left to `run-process-ownership` and `tender-pause-holds-reprompt`.

## To check by hand

Nothing below was run against two real machines.
Exercise adopt, a wake after a suspend, and a landing refusal on a second hub clone.

- A fixer commit (`ec2a3b5b6`) cleared the verify failures `cmd/lyx.TestCLIForm_EveryCommand`, `internal/battencli` and `internal/testkit/envkit`: owner seams made nil-legal in `envkit`, and the `lyx board upsert` and `lyx batten run` help kept under the length cap.
- `ReconcileOrigin`'s setowners pull-refusal inputs are not named, and its push-transport failure has no row (`internal/fabricengine`, setowners path).
- When the adopter lands first a run the old machine's Finalize had already merged into its local parent, the old parent needs a manual `git reset --keep` to origin's tip, or `git rebase --onto` dropping the squash.
  No `lyx` verb does that, and a Finalize re-entered through InnerRun's `Spawn` seam writes no halt note naming it.
  The old machine's pair is not torn down by a reconcile, and its records commit replays onto origin's parent records branch.
- A resume with origin unreachable trusts the local Bolt, so it can drive a run adopted elsewhere until the next owner check reaches origin; only Finalize's landing reconcile is certain to catch it.
- Two machines claiming an unowned prime while both are offline both act until one push wins; a start refused by its post-push re-read can leave a stopped run record that `lyx adopt <slug>` would resume as a takeover.
- A pair under a hub pause reads not live in adopt's liveness step, so `--drop-local` there can move a branch under interrupted driver strands.
- A run an older binary started has no owner until its next batten resume.
- A child holding the parent lock on a run with no live batten, or under an unlabelled pause, is not released until the next owner check.
- A driver still live on the old machine between adoption and its next owner check pushes into moved branches; those pushes are refused and recorded as failed pushes.
