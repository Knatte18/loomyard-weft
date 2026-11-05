```yaml
lens: board-scope
outcome: reported
```

Lens: board-scope. I checked the record's fit with the board entry, the touching notes, the peers and the parent commits.

**Findings**

1. MEDIUM, class `design`, D2 and D3: the record lets a member land into the beam after `lym beam ready`, which the entry rules out.
   - The entry says: "From `lym beam ready` on, the beam is held: no member lands into it until it has landed."
   - D2's bound covers a member filed between the ready record and the accept. When the re-check catches it, the hold is lifted and the dobby restarts, so that member runs and lands into the beam after ready.
   - The record says this case is "caught by the re-check, never landed over". The board's promise is stronger: no member lands at all.
   - Fix: either hold the beam from the ready record on, for example by having the dobby start no member run while the record stands, or state this as a departure from the entry.

2. MEDIUM, class `decision`, Open risks (#47 push lock): the entry asks for something the record does not do.
   - The entry's #47 section says: "a push lock whose holder died before a restart is taken as stale."
   - The record changes nothing there. It argues the premise is false, since a flock frees when its holder dies, and puts the case in Open risks.
   - This is a goal point the record leaves out because the entry may rest on a false premise. Only the parent can settle it.
   - Question for the parent: the stale-lock case cannot occur as written. Should lym stop a live push left over from before a restart, by some rule? Or does this point of the entry drop?

3. LOW, class `scope`, D9: the record goes beyond the entry.
   - The entry's Finalize point 4 is about the beam's pull request: Finalize keeps its title and body as the orch wrote them.
   - D9 makes Finalize stop editing any pull request it finds or has recorded, in every recipe, so task landings change too.
   - No entry rules this out and no other entry owns it, so it is not a conflict. It is task-run behaviour the entry does not ask for, and the record notes it in Open risks.
   - Fix: keep D9 to pawl, or accept the wider scope knowingly.

4. LOW, class `consistency`, D6: the record drops a verb the entry names, but the entry allows it.
   - The entry's verb list gives `lym beam pr open|close|merge` as an example. The record drops `merge`.
   - The entry also says the orch "never merges it", and the record lists the drop under Auto-mode assumptions.
   - No finding is needed; I note it only because the two texts differ.

**Goal points of the entry**

- Pawl becomes Preflight, Record-Write, Beam-Up, Beam-Await, Finalize, Beam-Down: met (D1).
- The orch opens the pull request when told, and its description lists every member with its pull request or landing commit: met (D5, D6).
- The parent comes in through a merge-in run filed by `lym beam merge-parent`: met. D3 refusal 3 names it and D14 tells the orch.
- Review with the orch; the pull request stays open while more members land; the orch updates the description: met (D6 update, D7).
- The operator can skip the pull request and land by a direct squash merge: met (D6 close, D8).
- `lym beam ready` ends Beam-Await, then Finalize and Beam-Down follow: met (D2, D1).
- `lym beam describe`, and `lym beam pr` open and close: met. `merge` is dropped, consistent with the entry (finding 4).
- `pr open` records an open pull request; closing it or the landing clears the record: met (D4).
- The dobby mails the orch at every member landing while the record stands, with no `notify_on_land`: met (D7).
- The orch never merges and never closes the pull request to add work: met (D6).
- Ready is the point of no return; `lym beam reopen` and the reroute go: met (D10).
- From ready on, the beam is held and no member lands: partly met (finding 1). The `RunHolds` boundary moves to Finalize: met (D2).
- Ready refuses when the parent holds commits not merged into the beam, naming merge-parent: met (D3.3).
- Ready refuses when the pull request is not mergeable now, read from GitHub's merge state, and `lym beam pr status` shows the same: met (D3.6, D6 status).
- Finalize steps 1 to 4 (parent lock, merge-in and gate; with a pull request, push and squash-merge on GitHub; without one, a local squash and push, Stuck naming a pull request; title and body kept): met (D8, D9).
- #57: the pull request stays open for added work, and Beam-Await survives a loop exit or restart: met (D6, D7, D11).
- #56: the title, body and `summary.md` are not rewritten, and describe lists each member's link, merge time and summary: met (D5, D9).
- #47: `lym beam start` brings the orch back during the landing: met (D12). The stale push lock: not met (finding 2). The ready flag is never withdrawn: met (D3, D10).
- In-flight runs are moved to Beam-Await and paused, the orch is told, and the pull request stays open; runs at Finalize or later carry on: met (D13).

**Entries, peers and parent commits checked**

- Note `pr-review-wait-silent` (B1): no conflict. The record scopes it out and removes only the pawl case.
- Note `dormant-orch` (F4): no conflict. The beam's orch is kept, as both entries intend.
- Note `beam-orch-root-project` (F19): no conflict.
- Note `reparent-running-task` (F20): no conflict. Its mention of Parent-Align is about task runs, not pawl.
- Peer `push-holds-remote-refs`: no conflict. Its record is a scaffold with empty Decisions. It is about the remote-refs lock, not `fabric.push.lock`, so it does not touch finding 2, and D3 and D6 only use that lock.
- Peer `quiet-notice-ignores-fixers`: no conflict. It overlaps D7 only as text in the dobby package.
- Peers `codex-trial3-faults`, `teardown-gone-pair` and `profiles-follow-template`: no conflict.
- Parent commits: `lym loom peers` lists none.

**Decision notes**

The board shows no `Decision for pawl-orch-landing:` note, and the record has no `Added after Discussion` entries. There was none I could not reach.

**Tool calls**

1. I listed the push-holds-remote-refs decision record and grepped it for push-lock text. My context held only that peer's title.
2. I grepped the same record for its headings and the packages it names. The first grep found nothing.
3. I read the same record with each line cut short, to confirm it is only a scaffold and has no decisions.
