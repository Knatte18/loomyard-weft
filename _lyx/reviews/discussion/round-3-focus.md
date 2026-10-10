---
round: 3
focus:
  - "Resume of a Done advisor: in decision-record D5 (an advisor is live while reed still tracks its strand; Done advisors are kept through KeepPane) and D8 (a resume probes each advisor with the chair's removing probe), settle what a resume does with an advisor whose run record carries a terminal outcome while its strand is still live, whether the chair is told it is dead, and what stops that strand at the chair's terminal result. Check that D5's definition of live and D8 agree, and that the acceptance criteria under Probe and resume cover that case."
  - "Advisor ends before the chair has a handle: in D7 (notices sent by Run.Send on the chair's handle) and D8 (advisors start first, the chair last, failed_advisors carries start failures), settle when each advisor's Wait join begins relative to the chair's start, and where an advisor death or timeout observed before the chair's handle exists goes. Check that every notice has a defined recipient and path."
  - "Concurrent senders into the chair's pane: in D7's Go-side notice paragraph, settle whether two notices, and a notice beside the chair's own Wait re-prompt or pending sends, can type into the pane at the same boundary, and whether the Decision states the serialization or why the overlap is harmless. Check the acceptance criteria for two near-simultaneous advisor ends."
  - "Chair-to-advisor authority: in D6 (an advisor's parent_directive rendered with the chair's strand name as the parent name, and advisor content never carrying authority), settle what a chair message can let an advisor do, what bounds it, and whether the spawned-run wording is stated as accepted or replaced. Apply the attack-surface question to this edge."
---

Each entry restates the sites it depends on, so a reviewer reads the current text of D5 to D8 directly.
