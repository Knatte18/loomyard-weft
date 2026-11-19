```yaml
lens: fix-proposal
outcome: reported
```

No findings.

Task: challenge the reviewer's proposals the fixer carried out as proposed (F1–F8, F10–F12; F9 was not fixed as proposed).

- F1: the one extra read after a timer wake with events still queued is bounded by the floor. The event/read race is covered, because the drain runs before the read.
- F2: a stale-board race is covered, because the watch loop's `BoardChanged` wakes the waiting poll.
- F5: no break found. `/` is not an API path, and the open verb's probe is no loss as keep-alive.
- F6: the page does not conflict with the new refusal. `send` at `board.html:829-834` omits `revision` when it is empty, and the create call at line 759 passes `''`.
- F7: checkRequest requires a revision for every other op, so a project slug cannot reach the empty-revision branch outside a create.
- F3, F4, F8, F10, F11, F12: no case found where the proposal is wrong or breaks a sibling.

Calls made:
1. Grep of `board.html` for the write payload and op sites: to see whether the page ever sends a revision on a create, which would break F6; my context held only the poll span.
2. Read of `board.html` lines 825-842 (the `send` function): to confirm that the payload omits `revision` when it is empty.
