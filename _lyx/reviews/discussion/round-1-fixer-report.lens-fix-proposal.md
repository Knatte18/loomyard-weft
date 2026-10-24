```yaml
lens: fix-proposal
outcome: reported
```

Fork for the fix-proposal lens: I challenged the reviewer's proposals the fixer carried out as written. 4 findings: 2 MEDIUM, 2 LOW.

**1. MEDIUM — D13's new sentence (from F13's proposal): "A caller with no run, such as hibernate's notify and fabric's push notifier, or one acting from the prime, notifies the prime's orch as today."**
- Summary: the "acting from the prime" exemption covers every batten notice, so batten's notices never take the walk that D13 lists them as taking.
- Evidence:
  - D8 says each batten verb "acts on the prime's location".
  - `internal/battencli/wire.go:569-573` (`rowNotices`) and `internal/battencli/drive.go:331-334` both call `orchcli.NotifyPrime(location, …)` with batten's location, which is the prime. Its comment reads "notices go to the prime's orch".
- Concrete case: a run in a beam with a live orch is "a caller acting from the prime", so its notices go to the prime's orch. That contradicts D13's list ("batten's notices at each delivery") and the D13 acceptance line "a run in a beam with a live orch reaches `ly:<beam>:orch`".
- Whether a caller has a run should decide it, not whether it acts from the prime.

**2. MEDIUM — D11's third refusal and its acceptance line (from F4's proposal): the refusal on a live beam orch strand or watcher, "so a landing leaves no orch strand or watcher behind".**
- Summary: the refusal is checked only at `lyx beam land`, so the stated guarantee doesn't hold.
- Evidence: D11 says "Batten's start guard applies the first two refusals to a project entry, and Worktree-Create applies them again". The orch refusal is not re-checked on a resume or before Worktree-Teardown.
- Concrete case: land passes the check, and the landing run spends time in Finalize's gate. Meanwhile the operator runs `lyx orch start` in the beam. D8 still admits that, since the beam has the mark and is owned here, and nothing refuses an orch start on a beam that is being landed. Teardown then ends the beam's reed session while the new orch's watcher, a detached `lyx orch watch` process (`internal/orchcli/spawn.go:30`), is registered nowhere teardown reaches.
- The acceptance line "leaves no orch strand or watcher of the beam" is therefore unproven. The reviewer's other option, land stopping the orch through orch's stop verb, or refusing `lyx orch start` in a beam whose entry is run-held by its landing, would close this.

**3. LOW — D2's rename refusal (from F9's proposal): "the refusal names `lyx beam land <slug>` for a beam … as the way forward".**
- Summary: the named way forward does not lead to a rename.
- Evidence: landing a beam ends it as a done project (D3: "a landed beam is a done project"), and D11 tears down its pair. A done project has no pair left whose name a rename could fix.
- What the operator is told to do destroys the thing they wanted to rename. The refusal should say a beam's slug cannot change while it lives.

**4. LOW — D1's new text (part of F12): the refusal "names the project and its members as what to depend on instead".**
- Summary: the suggested way forward can lead straight into another refusal.
- Evidence:
  - D2 lets notes be members, and the `workspaces` project holds notes.
  - D1 keeps "as no task may depend on a note today".
- Concrete case: a task depending on `workspaces` is told to depend on its members, which include notes. Writing that dependency is refused again. The way forward should name only the project's task members.

Tool calls beyond context:
- Read `internal/battencli/drive.go:320-335` and `internal/battencli/wire.go:565-576`, to confirm which location batten's notices are sent from; my context did not show batten's notifier wiring.
