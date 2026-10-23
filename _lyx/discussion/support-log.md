# Support log — verify-fix-escalation

## Interview

A darn run holds no interview: the writer read the board entry and the code and made each call itself.

## Rejected alternatives

- Running the cold fixer inside the gate closure, blocking the gate evaluation until it ends: it would block shuttle's wait loop on Merriam's strand for up to the recovery timeout, and spawning from the gate puts a second spawn site beside Master's own; the verb keeps Merriam as the one that acts on a gate message.
- Letting the tackler or the arbiter escalate after Webster ends stuck: neither fixes code, as the board entry states.
- A new `webster.yaml` model key for the cold fixer: the `recovery` key already names the stronger model, and a second key adds a knob no one has asked for.
- A separate full cold stencil duplicating the fork's fixing rules: two statements of the same rules drift; the shared body serves both.
- Escalating only after a fixer that made no commit: with a fixer that commits a wrong fix every time, the run would never reach the stronger model.
- Keeping the default of 3 attempts: the operator's view is that one fork attempt is enough before the stronger fixer.

## Review rounds

_No rounds yet._

## Question ledger

- Which attempt escalates? Self-picked: the last attempt the budget re-prompts, an attempt after a fixer that committed nothing, and every attempt after an escalated one.
- Should the default cap change? Self-picked: yes, to 2, following the operator's view.
- Where does the cold fixer write its reply? Self-picked: a reply file in Webster's scratch dir, outside `_lyx`, so the shared rule never to touch `_lyx` holds for both fixers.
- Should the verb refuse when the gate routed the fork? Self-picked: no; it refuses only without a verify-gate report, and the Master template limits when Merriam calls it.
