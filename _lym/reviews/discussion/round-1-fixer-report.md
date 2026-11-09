# Fixer report — discussion round 1

Every finding's premise held against the code; all are fixed in `_lym/discussion/decision-record.md`, with the support log's affected entries updated.

## Changes

- F1 — fixed differently. D10's `--here` reads the record first; an unfinished record that takes the worktree up makes `--here` take the record's own request, so it follows or resumes under D1 and D2.
  Not made as proposed: "`--here` is `lym adopt <subject>`" with the single subject, since D2's same-subject match would refuse a record over several subjects.
  D10's rationale stands; a criterion covers `lym s` after a worker killed at `deploy`.
- F2 — fixed as proposed. D4 covers the subject's worktree and every beam worktree reading its owner through `hubgeom.ParentOf` from the subject; criterion added for `lym adopt root`.
- F3 — fixed as proposed. `--discard` takes `adopt.lock` non-blocking, refuses naming `lym beam status` while held, and removes the record only holding it; the D2 criterion covers the race.
- F4 — fixed as proposed. A resume whose first step not done is `owner` or earlier starts at `reconcile`; the bound now reads "A resume past `owner`".
  The existing D2 criterion (killed after `pairs`, resumes at `pause`) contradicted this and was rewritten: killed after `owner` resumes at `deploy`, and killed after `pairs` re-runs the reconcile.
- F5 — fixed as proposed. The worker re-runs the caller's validation and D2's rules itself; the bound says a direct call is one more caller. Criterion added.
- F6 — fixed as proposed (first option). D10's bound states that `beam start` also judges the local Bolt and only warns, so the beam starts here, and the next reachable Bolt pull hands the subject to D11.
  The other option, `--here` failing offline, was rejected in the support log: it would stop every offline `lym s`.
- F7 — fixed as proposed (distinct-prefix option). D11's tags are `archive-owner/<slug>/<sha12>`, which `fabricengine.ArchiveTagSlug` and `remoteArchiveTags` do not parse; criterion added that the run report lists no run for them.
- F8 — fixed as proposed. D11's rationale quotes the rule text and cites the teardown's archive push for an `owned_elsewhere` verdict as precedent.
- F9 — fixed as proposed (bound option). D1's bound states that `lym fabric anchor`'s `runproc.EndSparing` ends the worker and D2's resume picks it up. Sparing the role was rejected in the support log, since the anchor would move the prime under a worker mid-step.
- F10 — fixed as proposed. D1 states the exit codes; criterion added.
- F11 — fixed as proposed. D2's record holds the end envelope, built from every step listed done.
- F12 — fixed as proposed, then amended by the self-check (see below).
- F13 — fixed as proposed. A fresh worker writes the record at `reconcile`; a resuming one rewrites only pid, start ticks, start and state.
- F14 — fixed as proposed. A call's local-commit flag replaces the record's other one; the D2 criterion covers it.
- F15 — fixed as proposed. D5's `pause` step skips slugs the record lists as paused; the Constraint and a D5 criterion follow.
- F16 — fixed as proposed. The record holds the resolved subject set, written with the record at `reconcile`; D4 tests against it.
- F17 — fixed as proposed. D5's bound states the old green end's full effect.
- F18 — fixed as proposed. D6 narrows the claim to the runs D5 pauses and states the front-task case.
- F19 — fixed as proposed (bound option). D11's bound states a failed owner pause lets the full pass go on as today.
- F20 — fixed as proposed.
- F21 — fixed as proposed, through F7's prefix: D12 lists only `archive-owner/` tags.
- F22 — fixed as proposed. D11 and D12 name each repository's origin and branch side.
- F23 — fixed as proposed. The Review criterion names the Dobby and orch pair rule's `--no-orch` exception.
- F24 — fixed as proposed. The D11 rationale line is deleted; the Auto-mode assumption stays.

Support log: rejected-alternatives entries for the offline `--here` refusal, the shared `archive/` prefix and sparing the adopt role; the stale `--here` and tag entries reworded; Q7 and Q8 updated; a round-1 line under Review rounds.

## Self-check

- fix-consistency, MEDIUM, D11 step 1 and criterion promise no pass while the bound allows one after a failed pause: fixed as proposed (step 1 and the criterion now speak of a run whose pause stands).
- fix-consistency, LOW, D11 rationale still argued for sharing `archive/` tags: fixed as proposed (sentence rewritten).
- fix-consistency, LOW, and fix-proposal, LOW (merged), D10's record match misses the beams D4 covers: fixed as proposed (D10 takes up a record listing a subject the worktree reads its owner from; criterion added).
- fix-consistency, LOW, support log Q7 stale: fixed as proposed.
- fix-proposal, MEDIUM, and fix-way-forward, MEDIUM (merged), a losing caller waits forever (a resume keeps the old start; a discard or a fast `done` leaves no record to follow): fixed with own change. A resume rewrites the record's start; the caller follows a later-started record whatever its state; a caller whose worker exited with none re-reads once and exits non-zero naming the re-run and `lym beam status`. Criterion added.
- fix-proposal, MEDIUM, and fix-way-forward, MEDIUM (merged), a held-back sibling blocks `lym s` on a fully adopted subject: fixed with own change. A record takes a worktree up only while its `check` step is not done; criterion added.
- fix-way-forward, LOW, an `interrupted` exit names no way forward: fixed as proposed (it names the re-run and `lym adopt --discard`).
- fix-claims: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
