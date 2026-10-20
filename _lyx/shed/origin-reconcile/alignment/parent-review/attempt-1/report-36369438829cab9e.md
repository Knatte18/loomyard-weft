---
verdict: APPROVED
---

# Alignment at parent-review — origin-reconcile

The design covers the board entry's goal and every design point, the three #737 bullets and the folded note `preflight-weft-push-sync`, with no work that belongs to another entry.

## Goal and design points

- Reconcile before what matters, two parallel `git ls-remote` calls over the warp and records repos, compared with tracking refs and Bolt statuses: D1, D2, D3.
- Call sites orch start, wake from `lyx hibernate`, batten resume, Merge-In and Finalize: D8; `lyx cleanup` is added from #737's "cleanup judges runs from a stale weft".
  Never per row or in a loop: the rejected alternative "Reconcile per shed row" and D8's closed caller list.
- Fast-forward: D4.
- Landed elsewhere, never resumed, local-only commits kept under `archive/*` tags and reported, pair removed locally with no push: D5, D6, D8's batten resume refusal.
  The entry also asks for the run record to be removed locally; D6 step 2 departs from that and states why: a local removal commit would be pushed over origin by the next push, against the entry's own "with no push".
  The prime weft instead reaches origin's state of the record through D5, and resume stays refused, so the entry's intent holds; the departure is a reasoned reading of the entry, not a scope deviation.
- Diverged stops and names the branch: D4 and D5, where a records branch stops only on a replay conflict, which reconciles the entry's "stop on divergence" with #737's rebase-and-drop-empties request, as D5's rationale says.
- `machine-handover`'s owner outcome slots in: D7.
- A rejected push fetches, rebases, retries and reaches the orch: D10.
- #737: `lyx cleanup --apply`'s empty `push_error` (D10), cleanup judging from a stale weft (D8), `lyx fabric pull` on a diverged weft (D9).
- Folded note: Preflight's push goes detached (D11); its open race question is answered in Scope Out.

## Other board entries

- `machine-handover` depends on this task; owners, `lyx adopt` and the owner outcome are left to it in Scope Out, and D7 gives it the slot its design point 4 expects.
- `run-process-ownership` depends on this task; orphaned processes and parked pushes under a pause are left to it.
  The local mode of `internal/pairteardown` is built here because this entry's landed-elsewhere removal needs it, and `run-process-ownership` reuses it; D5 and D9 cover part of its "`lyx fabric pull` reconciles stale status commits" expectation, and the support log names the remainder as left to it and `machine-handover`.
- `fabric-remote-async`: the reconcile's ls-remote and fetch of found heads are the synchronous exception that entry allows; D10's failed-push record answers its open question on how an async failure reaches the orch, and D11 is this entry's own folded note.
  No decision contradicts that entry.
- `umbrella-daemon`: dobby's calls of the reconcile are left to it in Scope Out.
- `tender-pause-holds-reprompt`: named as an open risk only.

## Peers and parent commits

- `lenses-module` (row 7, Plan-Write; own row 3): no overlap; its decision record touches config modules and lenses, nothing in fabric, pushes or the reconcile's call sites.
  No landing order applies.
- Parent commits: `lyx loom peers` lists none.
