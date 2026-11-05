```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the round-1 change set agrees with its surroundings (fix-consistency lens).

- **LOW.** Changed text: support-log.md, Rejected alternatives (new "created or opened" bullet) and the trimmed D1 Rationale. The existing Turn 3 line for board-scope 1 still says the departure was "fixed in D1's rationale", but D1's rationale no longer states it. Evidence: support-log.md line 36 reads "board-scope 1: the departure from "written when a hub is created or opened" was unstated — fixed in D1's rationale.", while D1's Rationale now reads only "no machine-wide list of hubs exists today, and the creel daemon is the one process every hub starts that mail needs anyway." The Round 1 F15 line records the move, but the earlier line was left as it was.
- **LOW.** Changed text: support-log.md Round 1 F13 line, and the dropped second Open risks bullet. Turn 3 still says this open risk was kept on purpose ("leanness: the `sandbox-closed-area-build` open risk duplicates D3's bound — rejected: the risk carries the broker route, a question for the parent that the bound does not raise"). The record has now dropped that risk. Without a cross-reference, the two lines read as a contradiction. Evidence: support-log.md line 41 against the new "F13: the `sandbox-closed-area-build` open risk dropped" line.
- **LOW.** Changed text: D3's new bullet "Any answer to `deliver` other than `accepted` or `refused`, an older binary's refusal of an unknown verb included, is a transient refusal the sending daemon retries (D2)." No acceptance criterion covers it. The record's convention is that every stated behaviour has one, and review finding F14 enforced that convention. Evidence: the D3 acceptance bullet gained a criterion for the pane-resolution half of F1 ("a `deliver` while a live strand … is accepted, not refused") but none for the unknown-verb or other-answer half. The D2 criteria cover only a `refused` answer returning at once.

Further tool calls: none.
