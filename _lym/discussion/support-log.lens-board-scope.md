```yaml
lens: board-scope
outcome: reported
```

Task: review the decision record's fit with the board (board-scope lens).

**Findings**

1. **MEDIUM**, D7 / Auto-mode assumptions, class `scope`. A pinned worktree waits out a limit even when another account has headroom, which contradicts the entry.
   - Evidence: the entry body, "The design (operator, 2026-11-15)", says "A usage limit is waited out only when no account of the machine has headroom."
   - D7 says "leaves a pinned worktree as it is", and the first auto-mode assumption says a pinned worktree "waits it out".
   - The entry makes no exception for pins. Either let a pinned worktree switch while its limit lasts, or put the departure to the parent as a question, since the pins are the operator's own design.

2. **MEDIUM**, D1 `account_pins.beams` / D9 / Open risks, class `scope`. The beams pin sends every session in a beam worktree to the pinned account, so the card implementer sessions there do not spread.
   - Evidence: the entry says pin "every orch and beam to one account, so all orch conversations under Remote Control land in one account online, while the worker sessions under them spread over the accounts".
   - The record admits the gap in Open risks ("does not spread") but does not design for it, so a stated goal point is left unmet.
   - Either pin by role (orch and dobby) instead of by worktree, or ask the parent whether workers in a beam worktree count as "worker sessions under them".

3. **LOW**, D3 / D9, class `consistency`. The record leaves the prime without a pick when adopt brings it onto a machine.
   - Evidence: the entry says the pick runs for "every worktree that adopt brings onto the machine", and `lym adopt root` adopts the prime.
   - D9 says "the prime has no entry until the operator sets one", while D3 picks for "each worktree `lym adopt` adds".
   - State whether `lym adopt root` picks for the prime. The entry's "The root worktree mostly stays on one account" fits either the beams pin or an explicit exception, but the record should say which.

4. **LOW**, D1 / D4, class `consistency`. With one account listed, D4 still exports it under a pin, so not everything runs as today.
   - Evidence: the entry says "A machine with one account, or none listed, runs everything as today with no pick."
   - D1 only says no pick runs with fewer than two accounts. A pin naming the one account still makes D4 export it and D5 add a status line.
   - It does follow the operator's own pin, so this is minor. The record should still say so.

**Goal points of the entry**
- Two or more accounts per machine, chosen by the operator: met (D1).
- One account per worktree, with a switch when needed: met (D4, D7).
- A pick by weight and by headroom in each window: met (D1, D3, D5).
- A pin for a whole repository: met through the hub pin (D1).
- A pin for every orch and beam, with workers spreading: partly met (finding 2).
- The root mostly on one account: met through the beams pin, but adopt is unclear (finding 3).
- A worktree whose account runs out switches, every spawn reads the account, the transcript copy is checked, and a limit is waited out only when no account has headroom: met, except for pinned worktrees (finding 1). The copy check rests on the API docs (D6), and the manual check is in D10.
- The policy lives in the machine layer and is never tracked: met (D1, D2).
- One account or none means no pick and today's behaviour: met, with finding 4.
- The pick runs on a new task, a new beam, and adopt onto a machine: met (D3).
- A move to another machine picks again there, and the adopt work owns how the run goes on: met (Scope Out, D3).
- The account is set once, a worktree with none runs as today, the switch is a verb, and git placement is weighed: met (D2, D3, D7).
- A size estimate is named: met (Constraints).
- The testing wiring for a second account is stated: met (D10).

**Entries, peers and commits checked**
- `adopt-build-resplit`, the dependency, is done. No conflict: D2 keeps nothing in git and D3 picks on adopt.
- `codexengine`, the Codex project: no conflict, since the record is Claude-only and parity is covered by its Constraints.
- `sandbox-closed-area-build` (note, labels claude and config): no conflict that I know of.
- `lym loom peers` lists no peers and no parent commits.
- Decision notes: there is no `Decision for claude-account-pick-simple:` note on the board, and none were unreachable.

**Tool calls**
- None. My context already held the board list, the task, and the peers output.
