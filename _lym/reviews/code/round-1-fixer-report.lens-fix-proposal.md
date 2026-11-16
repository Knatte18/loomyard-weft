```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals that the fixer carried out as proposed (F1, F3–F10). F2 followed the parent's ruling, so I skipped it.

**Findings**

1. **MEDIUM — F8, `remoteControlBarWordAt` (`internal/orchcli/remotecontrol.go`) and `recordRemoteControl`.**
   - The proposal draws no RC button when the record file doesn't name the strand.
   - Concrete case: an orch launched before this change, or after the record file or state was lost, runs on the claude engine with no record. The bar now draws no button, so the click cannot be reached. That is the case the board entry targets, a session "running without" remote control.
   - Before the fix such a strand drew a dim button. Only a relaunch brings the button back.
   - The same case arises when `adoptListedStrand` finds `st.Strand == ""`: `ReadRemoteControl` can never match it, so nothing is carried over.
   - The reviewer weighed only the Codex case.

2. **MEDIUM — F6, `adoptListedStrand` (`internal/orchcli/start.go`).**
   - The proposal copies `rec.On` from the record of `st.Strand` onto the adopted strand.
   - `adoptStrand` runs only when the state names a different run than the listed one. The old record describes the other run's session, not the adopted one.
   - Concrete case: the state names a strand whose record reads on. A different orch strand is listed live, started by hand or by an older build with no flag. The adopt carries on=true onto it, so the button lights green for a session that never had remote control.
   - A click then refuses with "already on, nothing was sent" (the F2 ruling). The only way forward is typing `/remote-control` by hand, so the button is wrong and its click is dead.
   - Carrying "off" or nothing is safe; carrying "on" is not.

3. **LOW — F7, `turnOnRemoteControl` (`internal/orchcli/remotecontrol.go`).**
   - The proposal returns success when the record write fails after the inject.
   - Concrete case: the write fails, so the record stays off and the button stays dim, but the command is typed.
   - The user clicks again because nothing lit, and the already-on guard doesn't fire because the record reads off. The command is typed a second time, which is the double-send F2 was ruled to prevent.
   - The click's success envelope also hides the failure. Only a Warn log shows it.

**Tool calls:** none. My context already held every changed file and the review.
