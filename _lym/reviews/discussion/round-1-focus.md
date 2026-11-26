---
round: 1
focus:
  - "D1, D3, D4 against the code: which runs the dobby counts as driven and on which ticks its steps run. Settle whether every stop in the Goal, landed and cancelled included, can still be read before the run leaves that set, its status files vanish or its state entry is cleared."
  - "D1 reading and episode stamp: settle against batten, the shed and `runfsm.Derive` that each layer's state values and history length behave as D1-D3 assume. Cover a second stop at the same row, a pause, a halt outside Run-Shed, a file-change-gated child read, and an unreadable or partly written status."
  - "D2 against D3: settle that 'a change into a stop' has one meaning for each transition under `stops`, such as stop to stop, a layer changing while another already stands at a stop, and batten's moves after the child's done."
  - "D3 landed and cancelled skip: settle whether the landing notice's memory is already set when the watch step judges that reading, given the order of the tick's steps, and what a run with `land_notice` on receives in each order."
  - "D5 delivery: settle that `orchcli.NotifyRunFrom` and creel can post with no topic and no run tag, keep order, survive the run's teardown, and report 'no orch strand' apart from a failed post. Check the retry, drop and hour bounds against D4's hold on the done-run clear."
  - "D6 inputs: settle that the slot holder record, `Pool.Holders`, the tier marker, `proc.PIDsWithEnv`, `proc.TreeSample` and `pauserecord.AwakeSince` supply what D6 rests on, including which worktree's gate config gives `timeout_min`, and what a nested or waiting gate hold looks like."
  - "D2, D6 new keys and attack surface: settle what each of `run_watch`, `gate_hold_factor` and `gate_hold_min` can silence or flood, and what bounds it, including negative or zero values and whether the hub keys are validated."
  - "Tool and behaviour facts: settle that each fact a Decision or its Rationale rests on states the observed result and where it is pinned, the suspend clock and the #216 cause included."
  - "Other board entries: settle whether `dotnet-gate-fixes`, `beam-member-parent`, `config-keys-nested` or landed parent commits, such as the background pusher and run-agent resume changes, contradict or already make a Decision."
---

Seed focus for round 1, set from the decision record, the support log and the writer's spec and stencil.
No code was checked.
