```yaml
lens: board-scope
outcome: reported
```

Task: check the decision record against the board (board-scope lens).

Findings

1. MEDIUM, class `scope`, Scope and D5: the brief's "log each delivery attempt" is covered only for hook claims. The board brief says "log each delivery attempt", and the title says mail "is logged". D5 logs the tool-boundary, Stop and ring claims. The record's Scope puts the pane path out (notices, reminders, injects and the typed bell), so those delivery attempts get no log line. Nothing in the entry limits logging to hooks. The record should either extend D5 to the pane and bell sends, or say in Scope why the existing journal (`read`, `delivered`, `injected` and `stalled` records) already counts as their log. As written, Out only says "no change to the journal", which leaves the brief's point half met without a reason.

2. LOW, class `consistency`, Constraints on `c49ba448`: the record replaces the landed commit's design decision ("the block is the only Stop output proven to keep the turn going") on evidence from the source code alone. `lym loom peers` lists no parent commits, so this is no conflict with a parent commit. The board entry asks for this change outright ("never label a delivery an error, and keep the command text out of the context"), so it is not a `decision` question. The record already names the commit and gives its reason, so nothing needs fixing beyond confirming that the tier3 acceptance criterion stays mandatory.

Goal points of the entry
- Mail reaches the model once: met (D1, D2, D3).
- Mail reaches the operator once: met (D1 drops systemMessage; D2 keeps one copy; D4 sets the summary label).
- One message, with no empty turn: met (D3).
- No error label: met (D1, D2, D4, with the bound that the label returns if the keys disappear).
- Hook command text kept out of the context: met (D1, D2).
- Scope "find why feedback plus blocking error": met (support log Turn 2, D1 rationale).
- Scope "find what writes `Stop says:`": met (D1, systemMessage).
- Scope "the ring hands the mail over and acknowledges on write": met (D3).
- Open "asyncRewake shown for sure": researched (D3 bound).
- Open "8 KiB budget": researched (Constraints).
- Open "ring races an operator turn": recorded as a risk.
- Open "model trust of additional context": recorded as a risk, checked by tier3.
- Brief "log each delivery attempt": partly met; see finding 1.

Entries, peers and commits checked
- T1 creel-mail-delivered-once (own entry): no decision edits it.
- T2 ide-seed-workspace-and-keys (the only peer, Darn-Write, no changed paths): no conflict.
- F4 landing-watch (creel-labelled note): no conflict; it adds a dobby-sent creel line and touches no hook delivery.
- Parent commits: `lym loom peers` lists none.
- Decision notes: no `Decision for creel-mail-delivered-once:` note exists on the board, so none was read and none was unreachable.

Tool calls
- `lym loom peers`: my context lacked the peers and parent commits.
- `lym board get landing-watch`: my context had only the brief of the one other creel-labelled entry, not its body.
