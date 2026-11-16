```yaml
lens: fix-proposal
outcome: reported
```

Task: the fix-proposal lens, challenging each reviewer proposal the fixer carried out as proposed.

Four findings: F5, F6 and F18 at MEDIUM, F7 at LOW.

**Findings**

1. **MEDIUM — F6 (D2 code-tip bullet and its criterion).** A crash between the leased push and the re-read leaves a given-up tip on origin.
   - Changed text: "pushes it ... with a lease on origin's code branch at the value it read with the records. After the push it reads the records tip again; when the run's last transition no longer names the tip, it resets origin's code branch with a lease".
   - The case: the owner gives the tip up, the owing machine pushes it, and its process dies before the re-read.
   - Origin's code branch then holds the given-up tip, and the reset happens only if the same process re-reads.
   - The owner's next code-side push builds on the earlier transition's tip, so it is non-fast-forward and rejected. The artifact gives that rejected code side no way forward.
   - The added criterion says the branch stays "where the owner's give-up left it". That holds only when the owing process survives.
   - The lease at the read value adds nothing here: a give-up writes only a records commit and never moves the code ref. The re-read alone carries the guarantee, and it is not crash-safe.

2. **MEDIUM — F18 (D5's new `PastParentMerge` bullet and the D5 Finalize criterion).** The fix lets a parent-branch push land under a hold.
   - Changed text: "`pairteardown.ReadRun` also sets `PastParentMerge` when the run's local parent branch holds its parent-side merge ahead of its upstream, so a Finalize cut ... pushes its parent branch under a hold".
   - Before the fix, the exemption covered only landings whose merge was already on origin. Those push only records and the board `done`.
   - The case: adopt step 3 fast-forwards the subject's branches and step 7 runs the check. Meanwhile a non-adopting machine pushes its cut Finalize's parent merge into that subject's branch.
   - The green check never saw that merge, yet step 8 clears the hold. The merge enters the hub unchecked, against D5's rationale that nothing happens until adopt's check is green.
   - It also changes the meaning of a landed input: `internal/runfsm/act.go:29` says "true when origin's fetched parent branch holds the task's work", and Scope Out says a decision "never redesigns" the landed FSM.
   - The review's other option avoids both problems: the run waits out the hold with its merge local.

3. **MEDIUM — F5 (D6 step 1's new line).** A re-run under this machine's own hold keeps that hold as it stands, whatever subjects the re-run names.
   - Changed text: "A hold this machine wrote stays as it stands ...: the commit writes only the owner and the registry entry, and the adopt goes on from step 2."
   - The case: this machine's `adopt` hold for subject A stands after A's adopt died. The operator runs `lym adopt B`.
   - The hold keeps naming A, and B's steps run. B's green check then "clears the hold" in step 8, though A's check never ran green.
   - `--take-hold` refuses a hold naming other subjects; the plain re-run path has no such guard.

4. **LOW — F7 (D3's machine id and the current-name refusal).** A machine whose `machine.json` is lost or recreated cannot get its own identity back.
   - Changed text: "A machine id is a random id lym writes into `machine.json` ... A registration whose current name is another registered machine's current name is refused the same way."
   - `machine.json` is per-user config (`internal/machinename/doc.go:2`), so a reinstall or a new user profile writes a fresh id.
   - The machine's own earlier entry then refuses its unchanged name. The only way forward offered is `lym machine name <new>`.
   - After that, every owner recorded under its old name reads as another machine's, so it must adopt its own subjects. No way forward reclaims the old entry.

Tool calls:
- Grepped `internal/machinename` for where `machine.json` lives, to build the F7 case; my context did not show the file's location.
- Grepped the `EndSubjectProcesses` doc, to test whether a hand-over mark written after the process end could kill its own caller; my context lacked that function. It never ends the caller, so that case yielded no finding.
