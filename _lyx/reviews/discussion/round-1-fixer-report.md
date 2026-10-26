# Round 1 fixer report — discussion

Every premise was checked against the code (`boardengine/project.go:102`, `loomcli/peers.go`, `dobbyengine/doc.go:58` and `template.yaml`, `loomcli/wiring.go:226-235`, `runproc/end.go`, `battencli/owner.go:241`, `adoptcli/deps.go:122`, `fabriccli/anchordeps.go`, `dobbyengine/loop.go:236-249`, commit fb5c10c3), and each holds.

## Changes

- F1, fixed: D9's hold bullet is now a rule over readers, not a closed list.
  Membership (`checkProjectField`) and `lyx loom peers` are named among its consequences.
  An acceptance criterion covers a task upserted into a beam at Beam-Await.
  The self-check narrowed the rule to readers that ask whether the entry is held from change; see the self-check below.
- F2, fixed: a new D9 bullet says a pawl run takes no `max_runs` slot while it holds no landing hold, and that the parent dobby still schedules its passes.
  An acceptance criterion covers it.
- F3, fixed: the hold is stated once in D9 over named rows: none from Worktree-Create through Beam-Await, one from Parent-Align on.
  Readers judge it from the producer half of the run status, and the bound for a failed board write is stated.
  D3's last bullet, D6 and D11 refer to the D9 rule.
- F4, fixed: the accept now stops the dobby first and runs the re-check after it, and Finalize re-checks `BeamLandRefusal` as well (the review's alternative).
  The self-check reworked the order further: the hold is written before the re-check, and a member's Worktree-Create reads the hold again.
  An acceptance criterion covers the race.
- F5, fixed: D6 resumes a beam's pawl run whenever this machine owns both the beam and its parent, whichever of the two the adopt named.
  The G criterion is reworded, and a criterion covers both adopt orders.
- F6, fixed: D6 skips the beam steps for a beam that holds the landing hold, still writes its owner, and names `lyx batten status <slug>`.
  An acceptance criterion covers it.
- F7, fixed: D4's last bullet names `lyx beam start` for both `lyx orch start` and `lyx prime`, citing fb5c10c3's full-command rule.
- F8, fixed: D1 says every `EnsureLive` caller starts only the beams it names: `lyx beam start`, adopt, and the anchor move, which restarts only the dobbies it ended.
  It also says none of them starts a dobby for a beam that holds the hold.
- F9, fixed: D4 states that `runproc` has no role-scoped end (`End`, `EndSparing` and `EndProcess` only) and gains one with `End`'s signal sequence and bound.
- F10, fixed: D5 says batten's start claim and its resume claim both write no owner, and that `resumeOwnerClaim`'s owner write goes.
- F11, fixed: in D7, a failed owner read counts as not owned elsewhere and is logged once per episode.
- F12, fixed: D12 checks on every board change the dobby ticks on, as well as when a run ends.
  The dobby's state record keeps the sent-notice episode.
- F13, fixed: the accept removes the request last.
  A re-entered Beam-Await posts the beam's orch an interrupted-landing notice naming `lyx beam start <slug>` and `lyx beam land <slug>`.
  The self-check tightened the notice's trigger.
- F14, fixed as proposed in D5.
- F15, fixed: D3's opening orders the refusals around the reconcile and the root claim.
- F16, fixed: D6 reads the entry's "There is no adopt of a beam".
- F17, fixed: acceptance criteria now cover the gone transcript, the re-armed can-land notice, and `lyx beam create`'s two ownership refusals.
- F18, fixed: D11 keeps only the requirement.
  The justification (different paths of one records branch, merged by the records push's replay) is in `support-log.md`'s Turn 7 walkthrough-1 line.

## Self-check

- fix-proposal, BLOCKING (the hold rule covered every reader, so adopt's started test, tender, batten's owner check and dobby stopped seeing the run): fixed.
  D9 now applies the no-hold reading only to readers that ask whether the entry is held from change, and says in-flight readers see the run at every row.
- fix-claims, MEDIUM, merged with fix-way-forward LOW (a failed re-entry write leaves `Parent-Align` standing, with no bound): fixed.
  Beam-Await writes its own status on entry and retries at each wake, and the bound in the hold bullet is rewritten to cover both of Beam-Await's crossing writes.
- fix-way-forward, MEDIUM (Finalize's cancel-then-run way forward halts again): fixed.
  The halt now carries `BeamLandRefusal`'s own per-member ways forward.
- fix-way-forward, MEDIUM (a late member that has already landed slips past Finalize): fixed with a different change.
  The accept writes the hold before the re-check, and a member's Worktree-Create re-reads the hold after making its pair, so either side sees the other.
  Finalize stays as a backstop with a stated bound.
- fix-proposal, LOW (Finalize framed only around hand runs; a member with no run): fixed by the same per-member ways forward.
- fix-proposal, MEDIUM, merged with fix-way-forward LOW (adopting a held beam without its parent strands the landing): fixed.
  The adopt report names `lyx adopt <parent>`, which resumes the landing by D6's resume rule, and the acceptance criterion matches.
- fix-proposal, LOW (a failed re-check restarts a dobby the operator stopped): fixed.
  The dobby is restarted only when this accept stopped it.
- fix-consistency, LOW (D11 omits Finalize as an applier of the land refusal): fixed in D11.
- fix-consistency, LOW (the interrupted notice fired after `lyx beam stop`, and its cause did not match the new order): fixed.
  The trigger is now no request plus a run status naming Parent-Align, and an acceptance criterion covers both cases.
- fix-consistency, LOW (`ensureParentDobby` restarts a held beam's dobby, contradicting D1): fixed.
  D1 names it among the starts a held beam refuses, and D4's bound says so.

Every fork returned a report.

## Disputed

None.

## Deferred

None.
