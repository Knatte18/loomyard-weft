# Fixer report — discussion round 1

Every finding's premise held against the code; all twelve are fixed in `_lym/discussion/decision-record.md`, with the support log's Rejected alternatives, Q6, Q8 and a new Round 1 entry updated to match.

## Changes

- **F1** (BLOCKING), fixed as proposed and widened: D8 gives `internal/machinecli` deps for the file path, the hubs to sweep and the working directory, so no verb test writes the developer's file or a real hub's Bolt (the working directory seam added because a test's directory lies inside this hub, which D6 sweeps).
  The claim that no outside test needs a change is replaced; a bound covers reading tests on a developer's bad file; a criterion pins the verb's tests to their seams.
- **F2**, fixed as proposed (first option plus bounds): D5 pulls each hub's Bolt through `fabricengine.Bolt.Pull` before the collision check; the remaining unseen cases (a push after the pull, a hub whose pull fails) are bounded.
  A new bound states that a shared name makes each machine read the other's entries as its own, with the way forward; the former-name exemption carries the same bound. The collision criterion now includes an origin-only owner.
- **F3**, fixed as proposed in its first form: the file's key is now `names`, holding every recorded name with the current one last, so a repair over a decodable file can go back to the last valid name. The undecodable case is stated, with a hand restore as its way forward; a criterion pins the repair.
- **F4**, fixed as proposed: hub-pause advice dropped from D6 with the code evidence; the support log's rejected alternative and Q6 updated.
- **F5**, fixed as proposed: D6's first recovery states that the owner pause ends the run's processes, and names the dobby's full pass at the lift as what drives the run again, citing `internal/tendercli/doc.go`, `internal/hibernateengine/doc.go` and `internal/dobbyengine/doc.go`. The hibernate-wake recovery is merged into it, since it has the same cause.
- **F6**, fixed as proposed (second option): the owner write selects owners in the replace set itself, after its pull; D6 no longer selects from `ReadOwners`. The replace-set criterion is rewritten to match.
- **F7**, fixed as proposed: D3 lists `machinename.FilePath`, `ReadFile`, `Names` and `CheckName`; `internal/machinecli` reads only through them; D7 reads former names through `machinename.Names` in the Landings seam; the Notes' scan pins that no production file outside `internal/machinename` names the file. A `Names` criterion is added.
- **F8**, fixed as proposed: `standalonestate.AbsoluteUserConfigDir` (new) ignores a relative `XDG_CONFIG_HOME` for the machine file, while `CredentialDir`'s rule stays; a D2 criterion pins it.
- **F9**, fixed as proposed: the Slack constraint states the rename-back case and the refusal's way forward.
- **F10**, fixed as proposed: D6 and its criterion name open beam projects only; a done entry keeps its owner.
- **F11**, fixed as proposed: D5 states that a landing under a lost name counts toward no board tidy, and gives its way forward.
- **F12**, fixed as proposed: D6 advises lifting a pause only after the hub's sweep reports done, citing the skipped deferred push; the review criterion names the advice.

## Self-check

- fix-consistency 1, LOW (Scope bullet still says one shared resolver): fixed as proposed.
- fix-consistency 2, LOW (D2 heading and rationale say one resolver): fixed as proposed.
- fix-consistency 3, LOW (no export gives the lock path): fixed with my own change. `machinename.FilePath` now returns the lock's path too, so `internal/machinecli` names neither file.
- fix-consistency 4, LOW (support log Turn 3 lines stale): fixed with my own change. I annotated both lines with what Round 1 changed instead of rewriting them, since Turn 3 is a record of that turn.
- fix-way-forward 1 (LOW) and fix-proposal 1 (MEDIUM), merged (the shared-name way forward blames the wrong sweep): fixed as proposed. D5 now names which machine to rename and says that a sweep holding the shared name in its replace set moves the other machine's entries, which each machine then adopts back.
- fix-claims 1, LOW (the tenderengine citation does not support the claim that no pause holds the orch's checks): fixed as proposed. D6 and the support log now cite the owner pause at bring-up in `internal/orchcli/doc.go`; there, a hub pause only delays the reconcile.
- fix-claims 2, LOW (the list of owner-pause sources reads as complete): fixed as proposed. The orch's bring-up and loom's pre-push are added, and the list is marked as not exhaustive.
- fix-claims 3, LOW ("counts it again" for a landing that was never counted): fixed as proposed.
- fix-proposal 2, LOW (a relative `HOME` or `LOCALAPPDATA` silently falls back to the hostname): fixed as proposed. `AbsoluteUserConfigDir` returns an error naming the variable, D3 and the verb surface it, and the D2 criterion pins it.
- fix-proposal 3, LOW (a pull that skips its fetch is not named as "fails"): fixed as proposed. D5 now covers a pull that returns an error or skips its fetch.

## Gate repair

- The citation gate flagged symbols and a package the task creates, cited without a ` (new)` mark: `standalonestate.AbsoluteUserConfigDir`, `machinename.FilePath`, `machinename.ReadFile`, `machinename.Names` and `internal/machinecli`.
  Each flagged citation in the decision record now carries ` (new)`.

## Disputed

None.

## Deferred

None.

## Rulings

None.
