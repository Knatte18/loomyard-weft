```yaml
lens: board-scope
outcome: reported
```

Board-scope review of `reed-status-buttons`: I found no BLOCKING or MEDIUM findings and one LOW.

**Findings**

- **LOW — D2 and D8, defaults in the template rather than the hub's config (class `consistency`).** The entry says "the hub's config holds the default set". The record ships the defaults in reed's embedded template, and the hub base only tunes them. Under `PATTERN-sparse-hub-base` this has the same effect, since a hub base holds only the units it tuned. The record could say so in one line so the wording does not read as a departure from the entry.

**Goal points of the entry**

- Buttons are config, not code; adding one is one entry with no build: met (D2).
- Grouped by place (`task`, `beam`), the prime takes `beam`, and a `prime` group replaces it there: met (D2, D8).
- `run` is a shell command run in the session's worktree, with `{slug}`, `{worktree}`, `{beam}`, `{session}` and `{strand}` filled at click time: met (D6, D7). The static values are filled at render time, which gives the same result. `{beam}` is defined as a path, an auto-pick the entry leaves open.
- The same row in every window, with `window: agents` and `{strand}` buttons only in agent windows: met (D5).
- `confirm` puts a y/n `confirm-before` prompt in front of the command: met (D7).
- A failing command shows a short status-line message and its output goes to a log: met (D7).
- The hub holds the default set and a machine-local override adds buttons: met (D3), with the LOW above.
- The default set holds HALT (`lym tender pause`), CANCEL behind confirm, VS Code, a terminal, a new agent, and distill and refresh behind confirm on the orch session: met (D8).
- Later buttons, such as `board-agent-button`, are entries too: met, and it is out of scope as it should be.
- Placement on a third line, `follow` moved there as a built-in, check mark left on line 2: met (D1).
- Open question on the new agent's role: settled as an auto-pick (`agent`, fixed). That is a choice the writer may make, not one for the parent.
- No decision edits this entry, files a task or promotes a note: met.

**Entries, peers and parent commits checked**

- `board-agent-button` (F2): no conflict. Its button becomes a later config entry, and the record leaves it out of scope.
- `dormant-orch` (F4): no conflict. The orch lives on per beam, so the distill and refresh buttons stay valid.
- `machine-switch` (peer): no conflict. Its parent ruled out a reed button of its own, and it touches no reed or desktop code.
- `gate-fewer-binaries-trial` (peer): no conflict, no overlap.
- `plan-split-producers` (peer): no conflict, no overlap.
- `lym loom peers` lists no parent commits.

**Decision notes**

The board list shows no `Decision for reed-status-buttons:` note, and the record has no `Added after Discussion` entries. Nothing was unreachable.

**Tool calls**

- `lym board get dormant-orch`: a touching orch entry that the inherited context had not read.
