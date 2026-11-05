# The beam orch drives the merge-in, pull request and review of its beam inside Beam-Await

`pawl-orch-landing` · Note · feature, pawl, landing, orch, workspaces

## Seen (operator, 2026-11-05)

The pawl recipe (`contracts/recipes/pawl-recipe.yaml`) runs Beam-Await, then Parent-Align, Beam-Describe and PR-Review as fixed producers before Finalize, as tapestry and darn do.
Tapestry and darn are autonomous runs with no session the operator talks to, so they need those rows.
A beam is a project: larger, iterated with the operator, and it has its own orch the operator talks to.
Fixed rows after Beam-Await get in the way: a PR-Review already started has to be cancelled, or reopened back to Beam-Await, when the operator finds the beam needs more work.

## Idea (operator, 2026-11-05)

The beam's orch drives the landing work itself, inside Beam-Await; pawl drops Parent-Align, Beam-Describe and PR-Review.
Pawl becomes Preflight, Record-Write, Beam-Up, Beam-Await, Finalize, Beam-Down.

1. Pull request: when the operator says so, the beam's orch opens the beam's pull request by the hub's rules for beam pull requests, with its description.
   The description lists every member of the beam that landed, each with its own pull request or landing commit.
2. Parent merge: the beam's orch brings the parent into the beam when the operator says so, through a merge-in run inside the beam (a darn run with its perch, as `lym beam merge-parent` files), so a hard merge gets a run of its own.
3. Review: the operator and the beam's orch handle the pull request's review together.
   If the beam is not ready, the pull request stays open: the orch files more member tasks, they land into the beam while it is open, and the orch updates the description (see below).
   The operator may also tell the orch to skip the pull request and land by a direct squash merge.
4. When the operator is satisfied, the orch runs `lym beam ready`; Beam-Await ends and Finalize lands the beam as today, then Beam-Down.

### Verbs for the beam's orch

The orch calls lym verbs, never `gh` by hand: thin verbs over the code the removed rows run today, for example `lym beam describe` (Beam-Describe's description writer) and `lym beam pr open|close|merge` (PR-Review's and Finalize's pull-request code).

### Members land while the pull request is open (operator, 2026-11-05)

Member tasks keep landing into the beam while its pull request is open, as was common and useful in Millhouse.
While the beam's pull request is open, every member task filed or running in the beam carries `notify_on_land`, so the beam's orch is told at each landing and updates the description, which keeps listing every member.

### `lym beam ready` checks the parent merge (operator, 2026-11-05)

`lym beam ready` refuses while the parent holds commits not merged into the beam, and its refusal names `lym beam merge-parent`.

## Built since (tidy, 2026-11-05)

Task `protected-parent-landing` landed (`dbd169ae`): `beam.yaml` `pull_request: merge` makes Finalize push the task or beam branch, find or open a pull request into the parent and squash-merge it on GitHub at once with no review, then pull the parent.
The review row returns Done for it (`MergeAtOnce`), and a parent push refused as protected is Stuck with a way forward naming `pull_request=merge`.

## Open points

- How Finalize lands a beam whose pull request the orch already merged on the remote; under a `merge` parent Finalize still squashes parent-side.
- Whether `notify_on_land` is set by lym when the pull request opens (and on each member filed while it is open), or by the orch.
- Resume of in-flight pawl runs that sit on a removed row.
