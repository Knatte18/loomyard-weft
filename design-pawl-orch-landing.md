# The beam orch drives the merge-in, pull request and review of its beam inside Beam-Await

`pawl-orch-landing` · Task · feature, pawl, landing, orch, workspaces, fabric · running · Plan-Write

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
`lym beam pr open` records that the beam has an open pull request, and closing it or the beam's landing clears the record.
While the record stands, dobby mails the beam's orch at every member landing into the beam, members filed after the pull request opened included, with no per-task `notify_on_land`; the orch updates the description, which keeps listing every member.

The orch opens, updates and reads the pull request; it never merges it and never closes it to add work.

### `lym beam ready` is the point of no return (operator, 2026-11-05)

`lym beam ready` is the last step of a beam: Beam-Await ends, Finalize starts, and there is no way back to Beam-Await, so `lym beam reopen` and PR-Review's reroute go.
From `lym beam ready` on, the beam is held: no member lands into it until it has landed (the hold boundary of `boardengine.RunHolds` moves from Parent-Align to Finalize).

`lym beam ready` refuses, naming the cause, while:

- the parent holds commits not merged into the beam; the refusal names `lym beam merge-parent`;
- the beam has an open pull request that GitHub would not merge now: a required review not approved, a required check failing or pending, or a conflict, read from GitHub's merge state for the pull request.
  A `lym beam pr status` verb shows the same state, so the orch checks it before `lym beam ready`.

### Finalize (operator and orch, 2026-11-05)

Finalize is the only place a beam is merged into its parent, under the parent lock and the landing gate:

1. It takes the parent lock, merges in what the parent gained since the last merge-in, and runs the landing gate over the merged tree, as today.
2. With an open pull request, it pushes the beam branch to it and squash-merges it on GitHub, then pulls the parent: the parent is typically protected against a direct push, so a beam with a pull request always merges on the remote, through the code `protected-parent-landing` built, which finds the pull request the orch opened.
3. When the operator told the orch to land without a pull request, it squash-merges into the parent and pushes, as today; a parent refusing the push is Stuck with a way forward naming a pull request.
4. It keeps the pull request's title and description as the orch wrote them, never overwriting them with a generated list (issue #56, `beam-landing-overwrites-pr-body`).

Then Beam-Down.

## Built since (tidy, 2026-11-05)

Task `protected-parent-landing` landed (`dbd169ae`): `beam.yaml` `pull_request: merge` makes Finalize push the task or beam branch, find or open a pull request into the parent and squash-merge it on GitHub at once with no review, then pull the parent.
The review row returns Done for it (`MergeAtOnce`), and a parent push refused as protected is Stuck with a way forward naming `pull_request=merge`.

## Folded in (issues #57, #56, #47; hub Models-LYMHUB, beam `duplication-and-import-followups`, 2026-11-04)

What these issues saw, and what this task does about each:

- #57: during a detour to add one more member, the beam's PR #267 was closed on GitHub, the pawl run failed at Beam-Await with `step Beam-Await interrupted (loop-exited)` so the ready flag was not read until a hand `lym loom resume`, and then blocked on the closed PR.
  Here: the pull request stays open for added work (above), and Beam-Await survives a loop exit or a restart: the next pass reads the ready flag with no hand resume.
- #56: after the orch reopened the PR with a hand-written title and a body listing every merged sub-PR with link, merge time and summary, the landing rewrote the title back and the body, and `_lym/landing/summary.md`, to a flat list of sub-PR titles.
  Here: Finalize keeps the title and body the orch wrote and never rewrites `summary.md` over the orch's text; `lym beam describe` builds the member list with each member's pull request link, merge time and summary.
- #47: after a restart during the landing, `lym beam start` refused to bring the beam's orch back ("held by its landing run"); a push hit `fabric.push.lock still held after 30s` from a push the restart left running; and no verb withdrew the ready flag.
  Here: `lym beam start` brings back the orch of a beam in landing, Finalize included, as the landing tolerates a live orch; a push lock whose holder died before a restart is taken as stale; a ready flag is never withdrawn, since `lym beam ready` is the point of no return, and the work that once needed a withdrawal now happens before it, in Beam-Await.

## In-flight pawl runs (operator and orch, 2026-11-05)

A pawl run that sits on Parent-Align, Beam-Describe or PR-Review when the new binary takes it is moved to Beam-Await, paused, and the beam's orch is told; an open pull request stays open, and the orch carries on with the new verbs.
A run at Finalize or later goes on as before.
