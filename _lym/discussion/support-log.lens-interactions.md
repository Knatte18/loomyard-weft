```yaml
lens: interactions
outcome: reported
```

Lens: interactions. I checked the pairs of decisions that touch the same state and the acceptance criteria against each one.

Pairs checked:
- D1 × D3 (status strand at key 1, runs strand at key 0, placed by D1's rule): held. Either add order gives runs first, then status. A status replace opens after runs. A runs replace finds no overview at key 0 or lower, so it opens before the lowest-index window.
- D1 × D4 (runs strand added beside status after the refusal is removed): held.
- D1 × D5 (an old status record reads as key 0 and ties with runs): held within D5's stated bound. Runs opens after the old status strand until the next loom start replaces status at key 1.
- D1 × D2 (several overview windows; fallback takes the lowest index): held. Under D3 and D4 the lowest index is always the runs window, or the only overview window.
- D2 × D3 (status moves from an own window, which counts as an agent window, to an overview window, which does not): held. Follow no longer jumps to the checklist. That matches D3's purpose.
- D3 × D5 (the replace from own window to overview at the next loom start): held. `ReplaceStrand` checks only `validateDisplay`; `refuseWindowMove` applies only to an update. The selection rule for a replaced overview moves a viewer to the new window.
- D3 × task sessions (status alone at key 1): held. With no overview window at key 1 or lower, it opens at the lowest index, as today.
- D4 × D5 (an older session where runs once replaced status): held. A stale runs strand is replaced at key 0, and status comes back at the next loom start.
- D1 × Resume or relaunch (strand-table order against tie order): broken for tied keys only; see finding 2.
- D1 × UpdateStrand (an order-key change on an existing overview strand): open; see finding 1.

Findings:

1. MEDIUM, D1. D1 says nothing about an update that changes the order key on an existing overview strand.
   - Evidence: `refuseWindowMove` (`internal/reedengine/strand.go`) refuses only anchor and window-group changes. An update that changes the new key would pass.
   - Result: the record changes, but the window stays where it is. That breaks D1's promise that overview windows "sit in ascending order key" until the next relaunch moves the window.
   - Fix: D1 should either make `refuseWindowMove` refuse a change to an overview strand's order key, or state the bound: the key is read only when the window opens.

2. LOW, D1 and its acceptance criterion "replacing either overview strand keeps the windows' order". Two rules in D1 disagree when keys tie.
   - D1 says tied windows sit "in the order their windows opened", and a reopened window goes after the last window whose key is not higher.
   - So relaunching the earlier of two tied windows moves it behind the other. Replacing the first of two tied windows does the same.
   - The criterion holds only when the keys differ. With D5's old key-0 status strand beside runs at key 0, a replaced runs strand ends up second.
   - D5's bound covers only the add order, not a replace or relaunch.
   - Fix: scope the criterion to distinct keys, and widen D5's bound to name replace and relaunch.

Constraints checked against the decisions:
- Reed stays generic: held. D3's title and key are told by the spawner.
- `PATTERN-named-constants`: held by D3.
- Window order after the overview windows: held by D1.
- Every acceptance criterion maps to its decision, apart from the tied-key case in finding 2.

Tool calls:
1. Read `refuseWindowMove` in `internal/reedengine/strand.go` and grepped `internal/reedcli` for the overview anchor. My context did not show what an update may change on an overview display, or whether reed's CLI exposes the anchor.
