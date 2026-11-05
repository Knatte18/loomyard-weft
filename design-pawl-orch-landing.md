# The beam orch drives the merge-in, pull request and review of its beam inside Beam-Await

`pawl-orch-landing` · Note · feature, pawl, landing, orch, workspaces

## Seen (operator, 2026-11-05)

The pawl recipe (`contracts/recipes/pawl-recipe.yaml`) runs Beam-Await, then Parent-Align, Beam-Describe and PR-Review as fixed producers before Finalize, as tapestry and darn do.
Tapestry and darn are autonomous runs with no session the operator talks to, so they need those rows.
A beam is a project: larger, iterated with the operator, and it has its own orch the operator talks to.
Fixed rows after Beam-Await get in the way: a PR-Review already started has to be cancelled, or reopened back to Beam-Await, when the operator finds the beam needs more work.

## Idea (operator, 2026-11-05)

The beam's orch drives the landing work itself, inside Beam-Await; pawl drops Parent-Align, Beam-Describe and PR-Review.

1. Pull request: when the operator says so, the beam's orch opens the beam's pull request by the hub's rules for beam pull requests, writing its description.
2. Parent merge: the beam's orch brings the parent into the beam when the operator says so, through a merge-in run inside the beam (a darn run with its perch, as `lym beam merge-parent` files), so a hard merge gets a run of its own.
3. Review: the operator and the beam's orch handle the pull request's review together.
   If the beam is not ready, the orch closes the pull request and files more member tasks.
   The operator may also tell the orch to skip the pull request and land by a direct squash merge.
4. When the operator is satisfied, the orch runs `lym beam ready`; Beam-Await ends and Finalize lands the beam as today, then Beam-Down.

Pawl becomes Preflight, Record-Write, Beam-Up, Beam-Await, Finalize, Beam-Down.

## Built since (tidy, 2026-11-05)

Task `protected-parent-landing` landed (`dbd169ae`): `beam.yaml` `pull_request: merge` makes Finalize push the task or beam branch, find or open a pull request into the parent and squash-merge it on GitHub at once with no review, then pull the parent.
The review row returns Done for it (`MergeAtOnce`), and a parent push refused as protected is Stuck with a way forward naming `pull_request=merge`.
This answers the open point on a protected parent below.

## Open points

- Verbs the beam's orch needs for 1 and 3, so the pull request and its description go through lym, not by hand: for example `lym beam pr open|close`, reusing Beam-Describe's description writer and PR-Review's pull-request code.
- How Finalize lands a beam whose pull request was merged on the remote; a protected parent is built (see above), and a pull request merged on the remote under a `merge` parent still squashes parent-side.
- The hold boundary: today the beam is held from Parent-Align on (`boardengine.RunHolds`); with an open pull request inside Beam-Await, whether member landings are refused while it is open, or update it.
- Whether `lym beam ready` still checks that the parent was merged in recently, since Parent-Align no longer runs before Finalize.
- Resume of in-flight pawl runs that sit on a removed row.
